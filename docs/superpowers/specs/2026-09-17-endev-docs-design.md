# endev-docs — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-review` (ver `2026-09-17-endev-review-design.md`) —
corre solo después de que aprobó (score ≥ 90).

## Objetivo

Dejar la documentación del proyecto — a nivel código (JSDoc o equivalente
según el lenguaje) y a nivel proyecto (README, `docs/`, o la convención
que ese repo ya use) — al día con lo que el plan efectivamente construyó.
Penúltima pieza de la cadena: `endev-plan` → `endev-execute` →
`endev-test` → `endev-review` → `endev-docs` → `endev-ship` (ver
`2026-09-17-endev-ship-design.md`), que es quien comitea y pushea todo al
final.

## No-objetivos

- No documenta nada que el plan no haya tocado — no es un barrido general
  de documentación del proyecto.
- No corre si `endev-review` no aprobó — documentar algo que todavía no
  pasó el gate de calidad es trabajo perdido si el fix cambia la forma.
- No inventa una convención de documentación nueva si el proyecto ya tiene
  una — sigue el patrón existente (ver paso 2).

## Flujo

### 1. Disparo

Se invoca automáticamente al final de `endev-review` cuando el score final
es ≥ 90. No se invoca manualmente sobre un plan que no pasó por el resto
de la cadena — necesita saber exactamente qué archivos tocó el plan.

### 2. Documentación a nivel código

Identifica el lenguaje del proyecto (sección Tech Stack del plan). Busca
la skill de documentación correspondiente:

- **JS/TS:** skill `jsdoc` (ya está en el repo).
- **Otro lenguaje sin skill de docs todavía:** reutiliza el mecanismo de
  `endev-execute` paso 2 (búsqueda de oficial → autoría desde documentación
  si no existe), con la misma pregunta de scope proyecto/global/temporal.
  No se reinventa una búsqueda distinta acá.

Con la skill ubicada, agrega o actualiza comentarios de documentación
(`/** ... */`, docstrings, etc.) en las funciones/clases/módulos
**públicos** que el plan creó o modificó — nunca código que el plan no
tocó.

### 3. Documentación a nivel proyecto

Antes de escribir nada, se detecta qué convención ya usa el repo:

- ¿Tiene `README.md` con secciones de features/uso? Se actualiza ahí.
- ¿Tiene una carpeta de reglas/documentación propia (ej. `.agents/rules/`,
  `docs/`)? Se sigue esa misma estructura en vez de crear una paralela.
- Si el repo no tiene ninguna convención de documentación de proyecto
  todavía, se pregunta al usuario dónde prefiere que viva antes de crear
  algo nuevo — no se asume una estructura de cero.

Se documenta únicamente lo que el plan agregó o cambió (qué hace el
feature, cómo se usa, decisiones no obvias) — no una re-narración de todo
el proyecto.

### 4. Estilo

`clinical-tone` + `anti-cliche` sobre toda la prosa nueva — sin relleno,
sin sicofantismo, sin clichés de redacción, igual que en el resto de la
cadena.

### 5. Commit

Se comitea (checkpoint, ver "Commits por tarea/ola" en
`2026-09-17-endev-execute-design.md`) con Conventional Commits (`docs:
actualizar documentación de <feature>`) y se agrega la entrada final a
`.plans/<tema>.state.json` antes de pasar a `endev-ship`.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| `endev-review` no aprobó (score < 90 tras las 5 rondas) | `endev-docs` no corre — se lo menciona en el resumen final como pendiente hasta que el review pase |
| El proyecto no tiene ninguna convención de documentación propia | Se pregunta al usuario dónde debe vivir antes de crear una estructura nueva |
| El lenguaje del plan no tiene skill de docs ni oficial ni propia | Se autora una (mismo mecanismo que `endev-execute`), con la pregunta de scope habitual |
| El plan no agregó ninguna función/clase pública nueva (solo tocó internals) | Se salta la documentación a nivel código; puede seguir aplicando la de nivel proyecto si corresponde |

## Testing (de este mismo skill)

Dry-run: correr la cadena completa sobre un plan de prueba con al menos
una función pública nueva y un cambio de comportamiento visible a nivel
feature, y confirmar que `endev-docs` agrega JSDoc a la función y actualiza
el README (o la convención que el proyecto de prueba use) — ambos acotados
exactamente a lo que el plan tocó.
