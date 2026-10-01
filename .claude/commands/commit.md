---
description: Crea un commit siguiendo la especificación Conventional Commits
argument-hint: [pista opcional: tipo, scope o contexto del cambio]
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git branch:*), Bash(echo:*)
---

## Contexto

- Estado: !`git status --short`
- Rama actual: !`git branch --show-current`
- Cambios staged: !`git diff --cached`
- Cambios unstaged: !`git diff`
- Últimos commits (para seguir el estilo): !`git log --oneline -10 2>/dev/null || echo "(sin commits todavía)"`

## Tarea

Crea **un commit** con los cambios actuales siguiendo [Conventional Commits 1.0.0](https://www.conventionalcommits.org/es/v1.0.0/).

Pista del usuario (opcional): $ARGUMENTS

### Formato

```
<tipo>(<scope opcional>): <descripción>

<cuerpo opcional>

<footer opcional>
```

### Tipos permitidos

- `feat`: nueva funcionalidad
- `fix`: corrección de un bug
- `docs`: solo documentación
- `style`: formato, sin cambios de lógica
- `refactor`: cambio de código que no corrige bug ni añade funcionalidad
- `perf`: mejora de rendimiento
- `test`: añadir o corregir tests
- `build`: sistema de build o dependencias
- `ci`: configuración de CI
- `chore`: tareas de mantenimiento que no tocan src ni tests
- `revert`: revierte un commit anterior

<!-- ### Reglas

1. La descripción va en imperativo, en minúscula, sin punto final y con un máximo de ~72 caracteres. Escríbela en el mismo idioma que los commits recientes (si no hay historial, en español).
2. El `scope` es opcional; úsalo si el cambio se limita claramente a un módulo o capa (p. ej. `services`, `repositories`, `routes`, `db`).
3. Usa el cuerpo solo si el *por qué* no es evidente; explica motivación, no un listado del diff.
4. Cambios incompatibles: añade `!` tras el tipo/scope (`feat(api)!: ...`) y un footer `BREAKING CHANGE: <descripción>`.
5. Si los cambios mezclan varias intenciones no relacionadas, propón dividirlos en varios commits y pregunta antes de continuar.
6. Si hay archivos staged, commitea solo esos. Si no hay ninguno, añade con `git add` los archivos relevantes por nombre (nunca `git add -A` a ciegas) y no incluyas secretos (`.env`, credenciales) ni archivos `*.db`.
7. Si no hay cambios, indícalo y no crees un commit vacío.
8. Nunca uses `--no-verify`, `--amend` ni `git push` salvo que el usuario lo pida explícitamente.
9. Termina el mensaje con esta línea de atribución, separada por una línea en blanco:

   `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>` -->

### Pasos

1. Si no hay cambios, díselo al usuario y detente.
2. Si no hay nada en staging, añade con `git add` los archivos relevantes por nombre
   (nunca `git add -A` ni `git add .`, y nunca archivos de secretos como `.env` o bases de datos como `data/*.db`).
3. Redacta el mensaje y usa la tool `AskUserQuestion` para preguntarle al usuario si el mensaje le parece bien.
4. Si el usuario acepta el mensaje, entonces ejecuta `git commit` pasando el mensaje con un HEREDOC. Y si no, redacta uno nuevo y vuelve al paso 3.
5. Ejecuta `git status` para confirmar el resultado y muestra al usuario el mensaje del commit.

### Ejecución

Pasa el mensaje con un heredoc para respetar los saltos de línea:

```bash
git commit -m "$(cat <<'EOF'
<tipo>(<scope>): <descripción>

<cuerpo opcional>

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>
EOF
)"
```

Al terminar, muestra el hash y el mensaje del commit creado (`git log -1 --oneline`).
