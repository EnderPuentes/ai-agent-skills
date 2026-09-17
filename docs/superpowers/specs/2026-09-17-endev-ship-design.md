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

## Mecanismo: reorganizar historial, no comitear desde cero

Versión anterior de este spec asumía que nada se comiteaba hasta acá, y
que `split-commit` operaba sobre un working tree crudo. Se corrigió
`endev-execute`/`endev-test`/`endev-goal`/`endev-docs` para que cada uno
comitee su propio checkpoint (necesario para poder bisectar un fallo en
medio de la cadena, que puede tardar varias rondas de `endev-review`). Eso
cambia el trabajo de `endev-ship`: en vez de agrupar un diff sin comitear,
**reorganiza un historial ya comiteado** en la cantidad de commits lógicos
que tenga sentido para el feature — vía `git reset --soft` al punto donde
arrancó la cadena (guardado en `.plans/<tema>.state.json`) seguido de
`git add`/`git commit` por grupo, usando el mismo criterio de agrupación
por responsabilidad de `split-commit` (que, igual que antes, está
diseñado para no comitear/pushear por sí mismo — `endev-ship` usa su
lógica de agrupación, no su modo de ejecución, y aplica esa agrupación
sobre commits existentes en vez de sobre un diff crudo).

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

### 2. Reorganizar el historial de la cadena

Con todo en verde: se lee de `.plans/<tema>.state.json` el commit desde el
que arrancó `endev-execute` (el checkpoint anterior al primero de la
cadena). Se hace `git reset --soft <ese-commit>` para volver todo el
trabajo de la cadena — los N commits de tareas/olas, los fixes de
`endev-goal`, el commit de `endev-test`, el de `endev-docs` — a staged sin
perder ninguno del historial previo a la cadena. Sobre ese diff total
(ahora sí sin comitear, pero acotado exactamente al trabajo de esta
feature) se corre el análisis de agrupación de `split-commit` y se ejecuta
cada grupo (`git add` + `git commit`) con sus mismas reglas de mensaje —
mismo criterio de siempre, aplicado una sola vez al final para que la
historia pública sea legible, sin haber sacrificado los checkpoints
durante la ejecución.

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
| Hay commits manuales del usuario intercalados durante la corrida de la cadena | El `reset --soft` a `chain_start_commit` los incluye igual (quedan después de ese punto) — `split-commit` los agrupa aparte por no matchear ningún patrón de tarea/fix/test/docs, pero si el usuario no los quería tocados, debería haber evitado comitear manualmente mientras la cadena corría |
| `chain_start_commit` no existe en `.plans/<tema>.state.json` (plan de una versión anterior a este cambio) | Se aborta y se pide correr `endev-execute` de nuevo para generar el manifiesto completo — no se asume un punto de partida |

## Testing (de este mismo skill)

Dry-run: correr la cadena completa sobre un plan de prueba, forzar a
propósito que ESLint falle (una regla violada a mano) y confirmar que
`endev-ship` se detiene sin comitear ni pushear. Corregir el lint y
repetir, confirmando esta vez que valida todo en verde, hace el `reset
--soft` exactamente hasta `chain_start_commit` (sin tocar historial previo
a la cadena), genera los commits lógicos esperados, y pushea.
