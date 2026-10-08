---
name: plan-issue
description: Lee una issue de GitHub, genera un plan detallado y lo publica como comentario en la issue. Recibe el número de issue como parámetro.
---

# plan-issue

Número de issue: $ARGUMENTS

Si `$ARGUMENTS` está vacío o no es un número válido, pregunta al usuario qué issue quiere planificar.

## 1. Leer la issue

1. Usa `gh issue view <número>` para obtener el título, descripción y etiquetas de la issue.
2. Guarda la información: número, título, descripción, URL, autor, fecha, etiquetas.
3. Si la issue no existe o no tienes acceso, informa al usuario y detente.

## 2. Analizar y explorar el código

Antes de generar el plan:

1. Explora el código relevante mencionado en la issue o relacionado con el dominio.
2. Familiarízate con la arquitectura del proyecto leyendo `CLAUDE.md` y los docs en `docs/`.
3. Identifica los ficheros que será necesario modificar y los patrones que seguir.
4. Determina si esta es una feature, fix, refactor, docs, chore o test.

## 3. Generar el plan

Crea un plan en markdown siguiendo esta estructura (adaptada del template en `newFeature`):

```markdown
# [Título de la issue]

| | |
|---|---|
| **Issue** | #<número> |
| **Estado** | Borrador |
| **Autor** | @<autor> |
| **Fecha** | AAAA-MM-DD |
| **Etiquetas** | <etiquetas> |

## 1. Contexto

[Resumen de 2-3 párrafos: qué problema hay, por qué importa, información de la issue]

## 2. Alcance

**Incluido**
- [Lo que se va a resolver]

**Excluido**
- [Lo que no entra y por qué]

## 3. Comportamiento esperado

[Describe qué debe pasar cuando se implemente, con escenarios concretos]

## 4. Diseño técnico

### Archivos afectados

| Archivo | Cambio |
|---|---|
| `ruta/archivo.ts` | Qué se modifica |

### Enfoque

[Patrón a seguir, precedentes, decisiones arquitectónicas]

### Modelo de datos / contratos

[Tipos, esquemas, o APIs nuevas/modificadas]

## 5. Casos borde y errores

| Situación | Comportamiento esperado |
|---|---|
| [Situación] | [Resultado] |

## 6. Plan de implementación

Pasos realizables en 5-10 minutos cada uno. Ejemplo:

1. [ ] Crear interfaz X en `ruta/archivo.ts` — arquitectura
2. [ ] Crear implementación base de X — `ruta/archivo.ts`
3. [ ] Añadir tests para X — `ruta/archivo.test.ts`
4. [ ] Integrar X en el servicio — `ruta/service.ts`
5. [ ] Tests de integración — `ruta/service.test.ts`

## 7. Criterios de aceptación

- [ ] [Criterio verificable]
- [ ] Tests pasan
- [ ] Código sigue convenciones de CLAUDE.md

## 8. Riesgos y consideraciones

- **Retrocompatibilidad:** ¿rompe algo?
- **Rendimiento:** ¿hay impacto?
- **Seguridad:** ¿datos sensibles o permisos?
```

Reglas al generar:

- Sé específico: nombra archivos, funciones, tipos reales del proyecto.
- Las tareas deben ser realizables en 5-10 minutos. Si alguna es mayor, divídela.
- Respeta la arquitectura definida en `CLAUDE.md`.
- Incluye tests como una tarea separada.
- Ordena las tareas para que el proyecto funcione tras cada una.

## 4. Mostrar y confirmar

1. Muestra el plan generado al usuario en un bloque de código markdown.
2. Pregunta si quiere:
   - Publicar el plan como comentario en la issue (recomendado).
   - Modificar algo del plan antes de publicar.
   - Cancelar.

## 5. Publicar como comentario

Si el usuario confirma:

1. Usa `gh issue comment <número> --body-file` con un fichero temporal para publicar el plan.
   - Guarda el plan en un fichero temporal.
   - Usa `gh issue comment <número> --body-file <fichero>` para publicar.
2. Verifica que el comentario se publicó correctamente.
3. Muestra al usuario la URL del comentario.

## 6. Almacenar localmente (opcional)

Si lo consideras útil para referencia:

1. Guarda una copia del plan en `docs/plans/<tipo>-<descripcion>.md`.
2. Informa al usuario dónde se guardó.

---

**Notas:**

- El plan es un borrador. El usuario puede editarlo o ampliarlo después.
- Si necesitas hacer cambios en el plan, pide confirmación del usuario.
- No comiences a implementar hasta que el usuario lo solicite explícitamente (eso es tarea de `newFeature`).
