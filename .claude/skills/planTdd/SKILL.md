---
name: plan-tdd
description: Genera un plan de implementación, lo implementa con TDD estricto y avisa por Slack al terminar el plan y al terminar el código. Todos los cambios se hacen en un git worktree aislado. Úsala cuando se pida una nueva funcionalidad, corrección o refactor que deba quedar planificada y notificada.
---

# plan-tdd

Tarea a resolver: $ARGUMENTS

Si `$ARGUMENTS` está vacío o es ambiguo, pregunta al usuario qué quiere hacer antes de continuar.

## Reglas inamovibles

- **Todos los cambios (plan, tests y código) se hacen dentro de un git worktree**, nunca en el árbol de trabajo principal.
- **Se avisa por Slack dos veces**: al terminar el plan y al terminar la implementación.
- TDD estricto: ningún código de producción sin un test en rojo que lo justifique.

## 1. Crear el worktree

1. Deduce `<tipo>` (`feature`, `fix`, `refactor`, `docs`, `chore`, `test`) y una `<descripcion>` corta en kebab-case, sin tildes ni espacios. La rama es `<tipo>/<descripcion>`.
2. Elige la rama base: `develop` si existe (local u `origin/develop`); si no, `main` o `master`.
3. Crea el worktree fuera del repositorio, en una carpeta hermana, y la rama a la vez:

   ```bash
   git worktree add ../<nombre-repo>-worktrees/<tipo>-<descripcion> -b <tipo>/<descripcion> <base>
   ```

   - Si la rama o la carpeta ya existen, avisa al usuario en lugar de sobrescribir.
   - Los cambios sin commitear del árbol principal no se tocan ni se arrastran al worktree.
4. A partir de aquí, **todas** las rutas y comandos (lectura, edición, `npm install` si hace falta, `npm test`) se ejecutan dentro del worktree. Si dispones de la herramienta `EnterWorktree`, puedes usarla para entrar en él; si no, usa rutas absolutas del worktree.
5. Si el worktree no tiene `node_modules`, ejecuta `npm install` en su raíz.

## 2. Generar el plan

Guarda el plan en `docs/plans/<tipo>-<descripcion>.md` **dentro del worktree**. Usa la estructura de `.claude/skills/newFeature/assets/TEMPLATE.md`, en español.

Antes de escribirlo, lee `CLAUDE.md` y los docs de `docs/` relevantes, y explora el código afectado para que las tareas sean realistas.

Reglas para las tareas del apartado "Plan de implementación":

- Cada tarea se puede implementar en **5-10 minutos como máximo**; si es mayor, divídela.
- Una tarea = un cambio verificable, con el test que lo cubre y los ficheros que toca.
- Orden tal que el proyecto siga funcionando tras cada tarea.
- Respeta el estilo arquitectónico del dominio tocado (hexagonal en `employee`, por capas en el resto) y las convenciones de `CLAUDE.md`.

### Aviso por Slack: plan terminado

Cuando el fichero del plan esté completo, envía un mensaje con las herramientas del MCP de Slack (`mcp__slack__*`, configurado en `.claude/mcp.json`; cárgalas con `ToolSearch` si son diferidas). Contenido:

- Título de la tarea y rama (`<tipo>/<descripcion>`).
- Nº de tareas del plan y ruta del fichero en el worktree.
- Una línea indicando que comienza la implementación.

Canal: `#planes-generales` (el mismo para los dos avisos), salvo que el usuario indique otro. Si la herramienta exige el ID del canal en vez del nombre, resuélvelo buscando el canal `planes-generales` con las herramientas de Slack. Si el envío falla (MCP no disponible, falta `BOT_SLACK_TOKEN`/`TEAMID_SLACK`, etc.), **dilo explícitamente al usuario** y continúa; no finjas que se envió.

Después de avisar, continúa con la implementación sin esperar confirmación, salvo que el plan contenga preguntas abiertas que bloqueen el trabajo: en ese caso, pregúntalas primero.

## 3. Implementar con TDD estricto

Para **cada** tarea, en orden:

1. **Red**: escribe primero el test. Ejecútalo y comprueba que **falla por el motivo correcto**. Si pasa sin código nuevo, el test no sirve: corrígelo.
2. **Green**: escribe el mínimo código de producción para que pase.
3. **Refactor**: limpia código y tests manteniendo todo en verde.
4. Ejecuta la **suite completa** (`npm test` en la raíz del worktree) y comprueba que pasa.
5. Marca la tarea en el plan (`- [ ]` → `- [x]`) **inmediatamente**, antes de la siguiente.

Notas:

- No avances si la suite no está en verde.
- Tests con Vitest (`*.test.ts` junto al código); servicios y casos de uso con los mocks existentes; repositorios SQLite contra una `Database` en memoria. Nunca contra `resttek.db`.
- Solo la API tiene tests. Si el cambio es de frontend, la verificación es al menos `npm run build -w @resttek/<app>`, y la lógica extraíble a la API o a código puro se prueba igualmente.
- Código, comentarios y mensajes de error en español según `CLAUDE.md`; identificadores según el idioma ya usado.
- Si cambias endpoints, roles, esquema o convenciones, actualiza el doc de `docs/` correspondiente (como una tarea más del plan).

## 4. Cierre y aviso por Slack

1. Ejecuta la suite completa una última vez (y el build del frontend si se tocó).
2. Comprueba con `git status` en el worktree que solo hay los cambios esperados (nunca `packages/api/resttek.db`).
3. **Aviso por Slack: implementación terminada**, por el mismo canal. Contenido:
   - Título, rama y ruta del worktree.
   - Tareas completadas (`x/y`) y resultado de la suite (nº de tests, pasa/falla).
   - Si algo quedó incompleto o fallando, dilo claramente en el mensaje; no lo presentes como terminado.
4. Resume al usuario lo hecho y dónde está el worktree. No hagas commit, push ni elimines el worktree salvo que el usuario lo pida (para commits está la skill `commit`; para limpiar, `git worktree remove <ruta>`).
