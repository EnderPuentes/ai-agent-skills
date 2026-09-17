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
- **Aserciones de accesibilidad obligatorias en todo test de UI:** cada
  test de Playwright que ejercita una pantalla o componente nuevo suma
  `@axe-core/playwright` (o la skill equivalente que la búsqueda de
  oficial de `endev-execute` ubique si el proyecto usa otro stack de
  testing) — contraste WCAG AA, focus-visible, focus trap en modales,
  `prefers-reduced-motion`. Sin esto, la cadena nunca detecta el tipo de
  falla que `clinical-tone` marca `[BLOCKING]` en su sección de UI/UX; un
  test que solo confirma "el botón funciona" no confirma que sea usable.
- Se corren todos los tests nuevos y se confirma verde antes de comitear.

### 4. Commit y handoff

Se comitea (checkpoint, ver "Commits por tarea/ola" en
`2026-09-17-endev-execute-design.md`) con Conventional Commits (`test:
agregar tests de integración para <feature>`), se agrega la entrada
correspondiente a `.plans/<tema>.state.json`, y se pasa a `endev-review`.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| No hay costuras de integración (el plan tenía una sola tarea) | Se salta este skill entero — no hay nada que un test de integración cubra que el unitario de esa única tarea no cubra ya |
| El framework de testing no se pudo resolver (docs no accesibles) | Se avisa y se continúa a `endev-review` sin tests de integración nuevos, dejándolo anotado en el resumen |
| Un test de integración nuevo falla | No se comitea con el test roto; se trata como un hallazgo más para `endev-review` en vez de bloquear acá |
| Falta assertion de accesibilidad y el proyecto no tiene skill de a11y ni oficial | Se autora una (mismo mecanismo de `endev-execute` paso 2), con la pregunta de scope habitual |

## Testing (de este mismo skill)

Dry-run: correr un plan con al menos dos tareas dependientes entre sí
(ola A produce algo que ola B consume) y confirmar que `endev-test`
escribe un test de integración que ejercita esa costura específica, lo
corre en verde, y lo comitea. Repetir con un plan que agregue un modal o
componente interactivo y confirmar que el test de Playwright generado
incluye aserciones de `axe-core` (contraste, focus-visible), no solo el
flujo funcional.
