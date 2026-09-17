# endev-ship — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-docs` (ver `2026-09-17-endev-docs-design.md`) — es
el último eslabón de la cadena.

## Objetivo

Validar el trabajo terminado (typecheck, lint, knip, tests) y, si todo
pasa, partirlo en commits lógicos y pushearlo — sin intervención manual.
Cierra la cadena completa: `endev-plan` → `endev-execute` → `endev-test`
→ `endev-review` → `endev-docs` → `endev-ship`.

## Desviación intencional de `split-commit`

El skill `split-commit` ya existente en este repo está diseñado
explícitamente para **no** comitear ni pushear — genera un script
copy-paste para que el usuario lo corra a mano ("Read-only git
inspection; no `git add`/`git commit`/`git push` por el agente", según su
propia descripción). `endev-ship` reutiliza su lógica de agrupar cambios
en commits lógicos (mismas reglas de mensaje, mismo criterio de
agrupación por responsabilidad), pero **ejecuta** esos commits en vez de
solo proponerlos, porque el usuario pidió explícitamente que esta etapa
final sea autónoma. Es una decisión consciente para esta cadena
específica, no aplica al uso normal de `split-commit` fuera de ella.

## No-objetivos

- No crea un Pull Request. Pushea a la rama actual contra su remoto ya
  configurado. Si el proyecto en cuestión requiere PR en vez de push
  directo, eso queda como paso manual — fuera de alcance de esta
  iteración.
- No corre si alguna validación falla — nunca comitea código roto.
- No re-valida nada que `endev-review`/`endev-test` ya confirmaron en
  verde; corre las validaciones de proyecto (typecheck/lint/knip) que
  esos pasos no cubren.

## Flujo

### 1. Validaciones

Se ejecutan, en este orden, las que el proyecto tenga configuradas
(se detectan en `package.json` — o su equivalente — antes de correrlas;
las que no existan se saltan sin marcarlo como fallo):

1. **TypeScript** (`tsc --noEmit` o el script `typecheck` del proyecto).
2. **ESLint** (o el linter que el proyecto use).
3. **Knip** (detección de código sin uso).
4. **Tests** (unitarios de `endev-execute` + integración de `endev-test`).

Si **cualquiera** falla, se detiene acá: no se comitea ni se pushea nada.
Se reporta qué falló, con el output relevante, y el trabajo queda tal
como estaba (sin commits nuevos) para que se corrija a mano.

### 2. Split commit (ejecutado, no solo propuesto)

Con todo en verde, se corre el análisis de `split-commit` sobre el diff
completo acumulado por toda la cadena — `endev-execute` (tareas e
integración de skills), `endev-test`, `endev-review`/`endev-goal` (fixes
de remediación) y `endev-docs` llegan todos **sin comitear** hasta acá (ver
"Sin commits al repo del proyecto" en `2026-09-17-endev-execute-design.md`)
— para agruparlos en commits lógicos siguiendo sus mismas reglas de
mensaje. A diferencia del uso normal del skill, acá se ejecuta cada grupo
(`git add` + `git commit`) en vez de imprimir el script.

### 3. Push

Se pushea la rama actual a su remoto ya configurado (`git push`, sin
`--force` ni cambios de rama).

### 4. Resumen final

Se muestra un resumen de la validación completa: qué chequeos corrieron
y su resultado, cuántos commits se crearon (con sus mensajes), y
confirmación del push (rama y remoto).

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| El proyecto no tiene `knip`/lint/typecheck configurado | Se salta ese chequeo puntual, no cuenta como fallo |
| Una validación falla | Se detiene todo el paso; no hay commit ni push parcial |
| La rama actual no tiene remoto configurado (`git push` sin upstream) | Se avisa y se pregunta si configurar el upstream (`-u origin <rama>`) antes de pushear, en vez de asumirlo |
| Hay cambios sin relación a la cadena mezclados en el working tree (de otro trabajo manual) | `split-commit` ya los separa en su propio commit lógico — no se mezclan con los de la cadena, pero si el usuario no los quería incluidos, debería haberlos stasheado antes de correr `endev-plan` |

## Testing (de este mismo skill)

Dry-run: correr la cadena completa sobre un plan de prueba, forzar a
propósito que ESLint falle (una regla violada a mano) y confirmar que
`endev-ship` se detiene sin comitear. Corregir el lint y repetir,
confirmando esta vez que valida todo en verde, genera los commits
lógicos esperados, y pushea.
