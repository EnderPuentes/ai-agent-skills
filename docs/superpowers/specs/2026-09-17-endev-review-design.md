# endev-review — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-test` (ver `2026-09-17-endev-test-design.md`) y,
transitivamente, `endev-plan`/`endev-execute` — necesita el plan original
completo (incluida la sección `## Acuerdos de grill-me`, ver
`2026-09-17-endev-plan-design.md`).

## Objetivo

Auditar la implementación terminada contra el plan **original** (no contra
lo que el código terminó siendo), puntuarla 0-100 sobre una rúbrica que
cubre seguridad, performance, escalabilidad, patrones de diseño,
modularidad, accesibilidad y vigencia del stack — no solo higiene de
UI/React — y si no llega a 90, generar y aplicar fixes puntuales hasta que
sí, sin desviarse del alcance del plan ni romper funcionalidad existente,
salvo que `grill-me` ya haya registrado un acuerdo explícito para ese
cambio.

**Por qué una rúbrica ampliada y no cinco skills separados:** sumar un
reviewer por eje (uno de seguridad, uno de performance, uno de
escalabilidad...) multiplicaría el costo de la etapa más cara de toda la
cadena (Opus, potencialmente varias rondas). En cambio, `/code-review` y
`security-review` — los dos únicos ejes con checklist curado ya
existente — se usan como insumo dentro de **una sola** pasada Opus, cuya
rúbrica explícita agrega los ejes que no tienen skill dedicado
(performance, escalabilidad, patrones, modularidad, vigencia). Un
reviewer senior real audita todo esto en una sola lectura del diff; este
diseño lo imita en vez de fragmentarlo en llamadas redundantes.

## No-objetivos

- No re-implementa el feature. Los fixes de remediación son puntuales
  (uno por hallazgo), no una reescritura.
- No decide sola aceptar un cambio de alcance — si un hallazgo requiere
  tocar algo fuera del plan y no hay acuerdo de `grill-me` que lo cubra,
  se detiene y pregunta en vez de decidir por su cuenta (ver "Errores y
  casos borde").
- No reemplaza revisión humana — el score es una señal, no una aprobación
  final; el resumen queda siempre disponible para que el usuario lo lea.

## Flujo

### 1. Preparar `.audits/`

Si `<proyecto>/.audits/` no existe, se crea. Si `.gitignore` del proyecto
no ignora `.audits/`, se agrega la línea. Los audits son artefactos de
trabajo, no documentación del proyecto — nunca se comitean.

### 2. Revisión (Opus, rúbrica de 7 ejes en una sola pasada)

Un subagente (`Agent`, `model: opus`) audita el diff completo que dejaron
`endev-execute` + `endev-test`, con el plan original (y su sección de
Acuerdos de grill-me) como referencia de alcance — no el estado actual del
código como si fuera la intención. Se apoya en dos skills como checklist
de insumo, y en una rúbrica explícita para el resto:

1. **`/code-review`** (o el que responda a ese nombre en el momento — hoy
   hay colisión potencial entre el built-in de Claude Code y el del repo
   `ai-agent-skills`, que asume comparación contra `develop`; usar el que
   esté activo y anotar cuál se usó en el reporte) — higiene UI/React.
2. **`security-review`** (built-in) — auth, superficie de inyección,
   secretos en código, XSS. Antes vivía como gate en `endev-plan` contra
   texto sin código; acá audita el diff real, que es donde corresponde.
3. **Rúbrica explícita para los ejes sin skill dedicado** (mismo subagente,
   mismo pase, sin invocación separada):
   - Performance: queries N+1, bundle innecesario en el path crítico,
     scripts bloqueantes, cache sobre queries caras repetidas.
   - Escalabilidad: supuestos de volumen/concurrencia que el plan declaró
     en `grill-me` — ¿el código los sostiene?
   - Patrones de diseño y modularidad: ¿siguió el patrón que `grill-me`
     acordó, o el que ya existía en el repo? ¿hay acoplamiento entre
     módulos que debería tener límites claros?
   - Vigencia del stack: ¿usa la versión estable actual detectada en
     `endev-plan` paso 1, o quedó código contra una API deprecada?
   - Accesibilidad: contraste WCAG AA, focus-visible, focus trap,
     `prefers-reduced-motion` — tomando como evidencia los resultados de
     `axe-core` que `endev-test` ya corrió, no re-analizando desde cero.

**Verificación de UI/frontend (cuando el plan la toca):** primero corre la
suite de Playwright que dejó `endev-test` (barato, sin costo de modelo por
interacción). Solo si Playwright no puede resolver la pregunta —
necesita juicio visual, o hay que debuggear un fallo intermitente
interactivamente — se recurre al MCP de `claude-in-chrome`. Nunca al
revés.

**Review incremental a partir de la ronda 2.** La primera pasada audita el
diff completo. Cada re-revisión posterior (tras un ciclo de `endev-goal`)
audita **solo el delta desde la ronda anterior** más los hallazgos que
seguían abiertos — no vuelve a leer el diff completo de cero. El diff
crece en cada ronda de remediación; re-auditar todo en cada vuelta es el
gasto de Opus más redundante de toda la cadena.

El subagente asigna un **score 0-100 holístico** (no una fórmula mecánica
sobre hallazgos) con su justificación, igual que lo haría un revisor
senior en un postmortem — más `clinical-tone` para el tono del reporte.

### 3. Guardar el reporte

Se escribe en `.audits/YYYY-MM-DD-<tema>-review-N.md` (N = número de
intento, empieza en 1). Incluye: score, hallazgos, cuáles ya estaban
cubiertos por un acuerdo de grill-me (y por lo tanto no cuentan como
desviación), y cuáles no.

### 4. Gate de 90 puntos

- **Score ≥ 90:** aprobado. Se muestra el resumen final al usuario, con
  link al reporte en `.audits/`. Fin del flujo.
- **Score < 90:** por cada hallazgo pendiente, se invoca `endev-goal` (ver
  abajo). Al resolver todos los hallazgos de esta ronda, se vuelve al paso
  2 para re-puntuar (intento N+1).
- **Tope de 5 rondas:** si después de 5 ciclos completos (goal → fix →
  re-review) el score sigue bajo 90, se corta el loop y se devuelve el
  control al usuario con el historial completo de `.audits/` — no sigue
  indefinidamente.

### 5. `endev-goal` — remediación puntual

Skill liviano, una responsabilidad: dado UN hallazgo específico del
review, arma y aplica un fix acotado a eso, sin más.

- **Entrada:** el hallazgo (del reporte de `.audits/`), el plan original
  completo (incluidos los Acuerdos de grill-me), y el diff actual.
- **Chequeo de alcance antes de tocar nada:** ¿el fix que este hallazgo
  pide se mantiene dentro de lo que el plan ya cubría, o rompe
  funcionalidad preexistente que el plan no tocaba? Si sí a lo segundo Y no
  hay un acuerdo de grill-me que lo autorice explícitamente → no se aplica
  el fix; se marca el hallazgo como "requiere decisión del usuario" en el
  reporte y se excluye del cálculo de la ronda siguiente hasta que el
  usuario decida.
- **Implementación:** un único subagente (`Agent`, `model: sonnet`)
  acotado solo a ese hallazgo — sin repetir el escaneo de skills ni el
  análisis de paralelismo de `endev-execute` completo; sería desperdicio
  para un fix puntual. Sigue el mismo ciclo TDD de `writing-plans` (test →
  falla → implementación → pasa) para ese cambio específico.
- **Salida:** commit del fix (`fix: <descripción puntual>`, checkpoint —
  ver "Commits por tarea/ola" en `2026-09-17-endev-execute-design.md`),
  entrada agregada a `.plans/<tema>.state.json`, vuelta a `endev-review`
  paso 2.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| Un hallazgo requiere romper funcionalidad existente sin acuerdo de grill-me | No se aplica; se marca "requiere decisión del usuario" y se pausa el loop para preguntar en vez de decidir sola |
| Se llega a 5 rondas sin superar 90 | Se corta, se devuelve el control con el historial de `.audits/` completo. El trabajo ya está comiteado por checkpoints (ver `endev-execute`) — no se pierde, pero tampoco llega a `endev-ship`; el usuario decide si mergea manualmente, sigue iterando, o descarta |
| `.audits/` ya existe pero no está en `.gitignore` | Se agrega la línea a `.gitignore` sin tocar el resto del archivo |
| El plan no incluye sección de Acuerdos de grill-me (plan viejo, previo a este cambio) | Se trata como "sin acuerdos registrados" — cualquier desviación de funcionalidad existente requiere pregunta al usuario |
| Dos hallazgos de la misma ronda tocan el mismo archivo | Se resuelven secuenciales (no en paralelo) dentro de esa ronda, para evitar que un `endev-goal` pise el fix del otro |

## Testing (de este mismo skill)

Dry-run: correr el pipeline completo sobre un plan de prueba, forzando a
mano que la primera revisión dé menos de 90 (ej. dejando un caso borde sin
cubrir a propósito), y confirmar que `endev-goal` lo resuelve, se
re-puntúa, y el segundo intento aprueba. Repetir forzando un hallazgo que
requiera romper algo preexistente sin acuerdo de grill-me, y confirmar que
el loop se pausa a preguntar en vez de aplicar el fix solo.

Forzar a propósito un hallazgo de cada eje nuevo por separado — una
query N+1, un componente sin `focus-visible`, una dependencia contra una
API deprecada — y confirmar que la rúbrica de la ronda 1 los detecta todos
en una sola pasada (no que hicieron falta llamadas extra). Confirmar
también que la ronda 2 en adelante audita solo el delta: medir que el
tiempo de la ronda 3 no crece proporcional al tamaño total del diff
acumulado.
