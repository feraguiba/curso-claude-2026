# CLAUDE.md

Guía para Claude Code al trabajar en este repositorio.

## Proyecto

Resttek: plataforma de gestión de restaurantes. Monorepo npm workspaces con 5 paquetes en `packages/`:

- `api` — Express 5 + TypeScript (ESM) + SQLite. Puerto 3000.
- `web-admin` (:4200), `web-empleados` (:4201), `web-clientes` (:4202) — Angular 21.
- `web-shared` — librería Angular compartida (auth, interceptores, login/registro, estilos). Se consume desde el fuente (`main: src/index.ts`), sin build.

Documentación de referencia en `docs/` (léela antes de cambios grandes):
- `docs/arquitectura/arquitectura-api.md` — capas, endpoints y roles, errores, alias.
- `docs/arquitectura/arquitectura-frontend.md` — patrón Store, auth, routing.
- `docs/dominio/modelo-datos.md` y `docs/dominio/glosario.md` — esquema SQL y vocabulario.

## Documentation

- `docs`
-- `api`
--- `openapi`
---- `openapi.yaml`: especificación de openapi de nuestra api.
--- `guidelines`
---- `api-pagination.md`: indica como paginar los resultados de las peticiones get.


## Comandos

Desde la raíz:

```bash
npm install            # instala todos los workspaces
npm run seed           # datos de prueba (idempotente, INSERT OR IGNORE)
npm run dev:api        # API con tsx watch
npm run dev:admin      # / dev:empleados / dev:clientes
npm test               # tests de la API (vitest run)
```

Dentro de `packages/api`: `npm run test:watch`, o un solo fichero con `npx vitest run src/services/order.service.test.ts`.

Build de un frontend: `npm run build -w @resttek/web-admin`.

Solo la API tiene tests; los frontends no tienen ninguno ni runner configurado. No hay linter. `web-clientes` tiene Prettier (`printWidth: 100`, comillas simples).

Credenciales del seed: la contraseña es el propio email (p. ej. `admin@resttek.com` / `admin@resttek.com`). Lista completa en el README.

## Backend (`packages/api`)

### Dos estilos arquitectónicos: respeta el del dominio que toques

- **Hexagonal + DDD solo en `src/contexts/employee/`** (`domain/` → `application/` → `infrastructure/`). `Employee` es la única entidad: constructor privado, `static create()` con validaciones, getters sin setters. Casos de uso = una clase con `execute()`. Interfaces con prefijo `I` (`IEmployeeRepository`). El cableado de dependencias está en `infrastructure/http/dependencies.ts`.
- **Por capas en restaurant, dish, ingredient, order**: `models/` (interfaces planas + `normalizeX()`), `repositories/` (interfaz sin prefijo `I` + `Sqlite*Repository` en el mismo fichero), `services/` (lógica y validación, varios métodos por clase), `controllers/`, `routes/` (aquí se instancian repo → service → controller).

No introduzcas entidades/casos de uso DDD en los dominios por capas ni al revés, salvo que se pida explícitamente.

### Convenciones

- ESM: los imports relativos y con alias **llevan extensión `.js`** (`import { dbConfig } from '@config/database.js'`).
- Usa los alias de `tsconfig.json`: `@config`, `@errors`, `@shared` (= `contexts/shared`), `@employee`, `@models`, `@repositories`, `@services`, `@controllers`, `@routes`, `@scripts`.
- TS muy estricto: `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `verbatimModuleSyntax` (usa `import type` para tipos).
- Estilo: 4 espacios, sin punto y coma, comillas simples.
- DTOs se declaran en el mismo fichero que el servicio/caso de uso que los usa.
- SQL: columnas en `snake_case`, renombradas a `camelCase` con alias en la propia consulta. `save()` hace UPDATE si existe e INSERT si no.
- Routers montados bajo `/restaurants/:restaurantId/...` necesitan `Router({ mergeParams: true })`.
- Nuevas rutas: monta en `src/app.ts` bajo `/api/v1`. Protege con `authenticate` y `authorize([...roles])` de `@shared/infrastructure/http/middlewares.js`.
- Roles válidos: `admin`, `manager`, `camarero`, `cocinero`, `cliente`. Los clientes son filas de `employees` con `role = 'cliente'` y `restaurant_id = null`.

### Errores

Lanza errores de `src/errors/DomainErrors.ts` (heredan de `AppError`). El `errorHandler` mapea por nombre de clase: los `*NotFoundError` listados → 404, `InvalidCredentialsError` → 401, resto de `AppError` → 400. Si creas un nuevo `NotFoundError`, añádelo a la lista del `errorHandler`. Los controladores delegan con `next(error)`; `OrderController` es la excepción (hace su propio `try/catch`).

### Base de datos

- `dbConfig` (`src/config/database.ts`) es la instancia compartida; API promisificada `run/all/get`.
- El esquema se crea en `runInitialMigrations()` con `CREATE TABLE IF NOT EXISTS`; no hay herramienta de migraciones. Cambiar una tabla existente no afecta a un `resttek.db` ya creado: hay que borrarlo y volver a ejecutar `npm run seed`.
- `PRAGMA foreign_keys = ON` está activo.
- No subas cambios de `packages/api/resttek.db`.

### Tests

Vitest con `globals: true`, ficheros `*.test.ts` junto al código. Los servicios y casos de uso se prueban con los mocks de `repositories/mocks/` y `contexts/employee/application/mocks/`; los repositorios SQLite contra una `Database` en memoria. Añade o actualiza tests al cambiar lógica de servicios, casos de uso o repositorios.

### Entorno

Variables: `PORT` (3000), `JWT_SECRET` (con valor por defecto de desarrollo, duplicado en `middlewares.ts` y `BcryptAuthService.ts`), `NODE_ENV`. `dotenv` está instalado pero no se carga en ningún sitio.

## Frontend (`packages/web-*`)

- Angular 21: standalone components, signals, **zoneless** (no añadas `zone.js`), guards/interceptores funcionales, lazy loading con `loadComponent`/`loadChildren`.
- **web-admin / web-empleados**: estructura `features/<feature>/{models,pages,services,store}`. Los componentes hablan solo con el **Store** (`providedIn: 'root'`, signals privadas expuestas con `asReadonly()`, trío `loading`/`error`/datos, `firstValueFrom` sobre el service). Tras crear/editar, actualiza la lista en memoria con `.update()`.
- **web-clientes**: modelos y servicios en `core/`; los componentes usan los services directamente con `.subscribe()`. El único store es `CartStore` (local). El componente raíz se llama `App`, no `AppComponent`.
- Los pedidos se refrescan por polling (no hay websockets): `OrderStore` en empleados (30 s), `my-orders` (10 s) y `order-detail` (5 s) en clientes. Limpia los intervalos al destruir el componente.
- Todas las llamadas van a `/api/v1` (relativo) a través de `proxy.conf.json`. En código nuevo, inyecta el token `API_URL` de `@resttek/web-shared` en vez de importar `environment.apiUrl`.
- Iconos Lucide: registra cada icono nuevo en `LucideAngularModule.pick({...})` del `app.config.ts` de la app correspondiente.
- Lo que compartan varias apps va en `web-shared` y se exporta en `src/index.ts`. Tras cambiar `web-shared`, reinicia el dev server del frontend.

## Flujo de trabajo

- Código, comentarios, mensajes de error y documentación en **español** (los identificadores siguen el idioma ya usado: inglés para clases y métodos, español para valores de dominio como `pendiente` o `cocinero`).
- Si cambias endpoints, roles, esquema o convenciones, actualiza el doc correspondiente de `docs/`.
- Antes de dar algo por terminado en la API, ejecuta `npm test`. En un frontend, al menos `npm run build -w @resttek/<app>`.
