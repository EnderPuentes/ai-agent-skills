# endev-test — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-execute` (ver `2026-09-17-endev-execute-design.md`) —
corre después de que todas las olas/tareas del plan terminaron.
**Precede a:** `endev-review` (ver `2026-09-17-endev-review-design.md`).

## Objetivo

Escribir los tests de **integración end-to-end** que cruzan el feature
completo del plan — algo que ningún worker individual de `endev-execute`
pudo escribir porque su contexto estaba acotado a una sola tarea.

## No-objetivos

- No re-escribe tests unitarios. Cada worker de `endev-execute` ya los
  escribió por tarea siguiendo TDD (formato `writing-plans`: test → falla
  → implementación → pasa).
- No re-escanea el código de cero asumiendo que la ejecución no cubrió
  nada — parte de que las tareas ya están implementadas y con sus propios
  tests unitarios en verde.
- No decide si el feature "está bien hecho" — eso es trabajo de
  `endev-review`. `endev-test` solo agrega cobertura; no audita.

## Flujo

### 1. Confirmar skill de testing

`endev-execute`, en su escaneo de gaps (paso 1 de su spec), ya debería
haber identificado y provisto el framework de testing del proyecto
(vitest, jest, Playwright, etc.) como una tecnología más a cubrir. Si por
algún motivo no está — plan generado antes de este cambio, o el gap no se
detectó — `endev-test` corre la misma resolución de `endev-execute` paso 2
inline antes de seguir (búsqueda de oficial → autoría si no existe).

### 2. Identificar costuras de integración

A partir del grafo de dependencias que `endev-execute` ya calculó (paso 5
de su spec: qué tarea consume qué produce otra), se identifican los puntos
donde dos o más tareas se conectan — esas costuras son exactamente lo que
un test de integración necesita cubrir, porque son las únicas partes del
sistema que ningún test unitario individual ejercita juntas.

### 3. Escribir y correr los tests

- Tests de lógica/API: siguiendo las convenciones de la skill de testing
  ya provista (ej. vitest + testing-library).
- Tests de flujo de UI/navegador (cuando el plan toca frontend): usando
  **Playwright** vía la skill `playwright-cli` ya disponible — scripts
  programáticos, sin costo de modelo por interacción. Cubren el camino
  feliz del feature completo y los bordes que el plan menciona
  explícitamente.
- Se corren todos los tests nuevos y se confirma verde (siguen sin
  comitear, ver paso 4).

### 4. Handoff

Sin comitear — igual que el resto de la cadena (ver "Sin commits al repo
del proyecto" en `2026-09-17-endev-execute-design.md`), los tests nuevos
quedan en el working tree y se pasa a `endev-review`. `endev-ship` los
incluye en el commit lógico correspondiente al final.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| No hay costuras de integración (el plan tenía una sola tarea) | Se salta este skill entero — no hay nada que un test de integración cubra que el unitario de esa única tarea no cubra ya |
| El framework de testing no se pudo resolver (docs no accesibles) | Se avisa y se continúa a `endev-review` sin tests de integración nuevos, dejándolo anotado en el resumen |
| Un test de integración nuevo falla | No se marca en verde; se deja tal cual (sin comitear, como todo lo demás) y se trata como un hallazgo más para `endev-review` en vez de bloquear acá |

## Testing (de este mismo skill)

Dry-run: correr un plan con al menos dos tareas dependientes entre sí
(ola A produce algo que ola B consume) y confirmar que `endev-test`
escribe un test de integración que ejercita esa costura específica, lo
corre en verde, y lo deja sin comitear para `endev-review`.
