# endev-plan — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)

## Objetivo

Un skill de Claude Code (`/endev-plan`) que actúa como "cerebro" planificador
para cualquier repo: reúne contexto de código con el menor costo posible,
somete la idea a interrogatorio (`grill-me`) cubriendo no solo ambigüedades
sino performance, seguridad, escalabilidad, patrones de diseño, modularidad
y vigencia del stack, redacta un plan de implementación con un modelo
fuerte (Opus), y lo hace revisar adversarialmente por un segundo modelo
(Fable) antes de guardarlo — salvo que el cambio sea lo bastante chico como
para no justificar el circuito completo (ver "Camino rápido").

## Alcance de esta cadena: una feature por corrida

Cada corrida de `endev-plan` → ... → `endev-ship` procesa **una feature**,
de punta a punta, de forma secuencial. El paralelismo de tareas *dentro* de
una feature (las olas de `endev-execute`) es un eje ya diseñado y no
cambia. El paralelismo *entre* features (correr dos cadenas completas al
mismo tiempo) queda **fuera de alcance** de esta iteración — se evalúa más
adelante, no es parte de este diseño.

## No-objetivos

- No reemplaza `brainstorming`/`writing-plans` para tareas simples — sigue
  siendo válido invocarlos directo cuando no se necesita el circuito
  multi-modelo.
- No ejecuta el plan. Solo lo produce; la ejecución se delega a
  `endev-execute`.
- No gestiona autenticación/créditos de `opencode` ni `ollama` — asume que ya
  están configurados en la máquina (verificado: `opencode` 1.18.31 con
  proveedor "OpenCode Zen"; `ollama` con `qwen3-coder:latest`).
- No corre paralelismo entre features — ver "Alcance" arriba.

## Convención de nombres

Todo skill o subagente custom creado de acá en adelante usa el prefijo
`endev-` (ej. `endev-plan`). Distingue lo propio de skills de terceros
(`grill-me`, `anti-cliche`, etc.) instalados en el mismo repo.

## Ubicación e instalación

- Vive en este repo (`ai-agent-skills`) como `endev-plan/SKILL.md`.
- Se enlaza a `~/.claude/skills/endev-plan` (symlink), igual que el resto de
  los skills de este repo.
- Se documenta en el `README.md` del repo junto a los demás skills de
  Productivity.

## Flujo

### 1. Contexto de código (cadena de fallback)

Objetivo: minimizar costo de tokens de pago en la fase de exploración.

1. **`opencode run "<consulta de búsqueda>"`** contra el repo actual
   (proveedor configurado: OpenCode Zen). Se considera fallo: código de
   salida distinto de 0, stderr con mención a créditos/cuota/auth, o timeout
   (60s).
2. Si falla → **`ollama run qwen3-coder`** local, mismo tipo de consulta.
   Se considera fallo: `ollama` no responde, o el binario no está en PATH.
3. Si falla → subagente **Haiku 4.5** (`Agent` con `model: haiku`) que hace
   la misma búsqueda con Grep/Read/Explore.

Cada transición se anuncia en una línea corta ("usando opencode para
explorar el repo" / "opencode no respondió, cayendo a ollama local" /
"ollama no disponible, cayendo a un subagente Haiku"). El resultado de
cualquiera de los tres niveles alimenta el mismo resumen de contexto; el
resto del flujo no distingue de dónde vino.

**Caché del fallo de opencode:** si opencode falla por créditos agotados
(no por timeout de red puntual), se guarda un flag `opencode_degraded: true`
con timestamp en `~/.claude/.endev-state.json` (fuera de cualquier repo,
es estado de la máquina). Mientras ese flag esté activo, las corridas
siguientes saltan directo a `ollama` sin pagar los 60s de timeout — se
reintenta opencode automáticamente después de 1 hora, o antes si el
usuario lo pide explícitamente.

**Chequeo de vigencia del stack:** como parte de este mismo paso, se
confirma la versión estable actual de las dependencias clave que el plan
va a tocar (vía el registro del paquete — npm/PyPI/crates.io según
corresponda — o la propia doc oficial) en vez de asumir lo que el modelo
ya sabe por entrenamiento. Esto alimenta tanto a `grill-me` (paso 2) como
al criterio de "sincronización con lo nuevo estable" que se aplica en todo
el resto de la cadena.

**Manifiesto de corrida:** se crea `.plans/<tema>.state.json` con
timestamp de inicio, el nivel de contexto usado (opencode/ollama/Haiku), y
un slot vacío por cada etapa siguiente de la cadena. Cada skill posterior
(`endev-execute`, `endev-test`, `endev-review`, `endev-docs`, `endev-ship`)
le agrega su propia entrada (modelo usado, resultado, timestamp) en vez de
dejar el estado de la corrida repartido en media docena de documentos
Markdown sueltos. Es el mecanismo real de trazabilidad/debugging de toda
la cadena — sin esto, reconstruir qué pasó en una corrida fallida significa
releer cada `.md` a mano.

### 2. Interrogatorio (`grill-me`)

Se invoca el skill `grill-me` con el contexto ya reunido, para resolver
ambigüedades del plan una pregunta a la vez con el usuario, igual que lo
haría `brainstorming` en su fase de clarificación — pero apoyado en lo que
ya se exploró del código, no solo en la descripción del usuario.

**Rúbrica de interrogatorio ampliada.** No se le pasa a `grill-me` solo la
ambigüedad funcional — se lo prima con un checklist de ejes que, si no se
preguntan ahora, se descubren recién en `endev-review` (demasiado tarde,
ya con código escrito): performance esperada (volumen, latencia
aceptable), seguridad (superficie de datos sensibles, auth), escalabilidad
(¿esto necesita soportar crecimiento o es de uso acotado?), patrones de
diseño y modularidad (¿hay un patrón ya establecido en el repo que este
feature debería seguir?), y vigencia del stack (¿la versión estable
detectada en el paso 1 es la que se debe usar, o hay una razón para fijar
una versión distinta?). No todos los ejes aplican a todos los planes —
`grill-me` los explora solo donde el contexto del paso 1 sugiere que son
relevantes (mismo principio que ya usa para no preguntar lo que el código
ya responde).

**Cada acuerdo que implique tocar funcionalidad existente o desviarse de un
supuesto por defecto se registra textual en una sección `## Acuerdos de
grill-me` del plan final** (paso 7). Esto es lo que
`endev-review` (ver `2026-09-17-endev-review-design.md`) usa después para
distinguir un cambio autorizado de una desviación real del plan.

### 3. Camino rápido (sin Opus/Fable ni contexto escalonado)

Si de la interrogación del paso 2 surge que el cambio es acotado — un
archivo, sin tecnología nueva, sin cruce de módulos, sin implicancia de los
ejes de arriba — se salta el resto de este flujo: no hay borrador Opus, no
hay validación Fable, y el contexto del paso 1 no necesita más que una
búsqueda directa (Haiku, sin escalonar opencode/ollama). Se redacta el plan
directo en formato `writing-plans` estándar y se pasa a `endev-execute` sin
más ceremonia. Esto evita que un fix de una línea pague el mismo costo que
una feature multi-archivo — es la causa más grande de latencia
injustificada en esta cadena si no se aplica.

### 4. Borrador (Opus)

Un subagente (`Agent`, `model: opus`) redacta el plan completo siguiendo el
formato de `superpowers:writing-plans` (header, contexto, tareas
bite-sized, testing) usando el contexto del paso 1 y las respuestas del
paso 2.

### 5. Validación adversarial (Fable)

Un subagente (`Agent`, `model: fable`) recibe el borrador y lo audita
buscando huecos, riesgos no contemplados y supuestos débiles — no reescribe,
solo señala. Opus incorpora los ajustes válidos. Una sola ronda (sin loop).

**Manejo de fallo:** si la llamada a Fable falla, se reintenta una vez. Si
vuelve a fallar, el plan de Opus queda como final sin validación cruzada, y
el documento final incluye una nota explícita: "Validación Fable omitida
(fallo de API tras reintento)".

### 6. Estilo (`clinical-tone` + `anti-cliche`)

Se aplican ambos skills sobre la prosa final del documento (no sobre el
código de ejemplo que el plan pueda incluir) para eliminar relleno,
sicofantismo y clichés de redacción.

**Nota sobre seguridad:** ya no hay un gate de `security-review` acá. Los
riesgos de seguridad que el contexto del paso 1 detecta (auth, secretos,
RBAC, red, datos sensibles) ya se resolvieron como preguntas de `grill-me`
en el paso 2 — correr `security-review` sobre texto de un plan sin código
real es una adivinanza y duplica trabajo. La validación de seguridad real,
contra el diff que efectivamente se escribió, pasa a `endev-review` (ver
`2026-09-17-endev-review-design.md`), que es donde hay código para
auditar.

### 7. Guardado

El plan final se escribe en `.plans/YYYY-MM-DD-<tema>.md`, relativo a la
raíz del repo donde se invocó el skill (se crea `.plans/` si no existe).
Este es un override explícito del usuario sobre el default de
`writing-plans` (`docs/superpowers/plans/...`).

### 8. Handoff de ejecución

**Actualizado (ver `2026-09-17-endev-execute-design.md`):** en vez de
ofrecer `subagent-driven-development`/`executing-plans` directamente, se
ofrece invocar `endev-execute <ruta-del-plan>`, que audita/provee las
skills necesarias para el plan (oficiales o nuevas, deprecando las propias
que quedaron obsoletas) y recién entonces dispara una de esas dos.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| `opencode` sin créditos/timeout | Cae a `ollama` (ver paso 1) y marca `opencode_degraded` |
| `opencode_degraded` activo | Se saltea opencode directo a `ollama`, sin pagar el timeout, hasta que expire la marca (1h) o el usuario fuerce un re-chequeo |
| `ollama` no disponible | Cae a subagente Haiku (ver paso 1) |
| Los tres niveles de contexto fallan | Se avisa al usuario y se pregunta cómo seguir (no se asume contexto vacío) |
| Fable falla tras reintento | Plan queda solo con Opus, se anota en el documento |
| El repo actual no tiene `.plans/` | Se crea |
| El plan califica para camino rápido (paso 3) | Se saltea Opus/Fable y el contexto escalonado; plan directo con `writing-plans` |

## Testing

Los skills son instrucciones, no código — no hay suite automatizada. La
validación es una corrida real de dry-run: invocar `/endev-plan` sobre una
tarea chica y real (por ejemplo, un cambio menor en `my-wallet-app`) y
confirmar en la transcripción:

- Que la cadena de fallback de contexto se anuncia correctamente.
- Que `opencode_degraded` evita el timeout de 60s en la segunda corrida
  consecutiva tras un fallo por créditos.
- Que `grill-me` pregunta por los ejes ampliados (performance, seguridad,
  escalabilidad, patrones, vigencia) solo cuando el contexto los hace
  relevantes — no en cada plan sin distinción.
- Que un cambio de un solo archivo sin tecnología nueva toma el camino
  rápido (paso 3) y nunca invoca Opus/Fable.
- Que un cambio real multi-archivo sí produce un borrador Opus y una
  crítica Fable.
- Que el documento final aparece en `.plans/` con el formato esperado, y
  que `.plans/<tema>.state.json` se crea con la primera entrada.
- Forzar el fallo de Fable (ej. desconectando de internet) para confirmar
  el mensaje de fallback en el documento.

## Alcance de esta iteración

Un solo sub-proyecto: crear `endev-plan/SKILL.md` y enlazarlo. No incluye
cambios a `grill-me`, `clinical-tone`, `anti-cliche` ni a `writing-plans` —
se consumen tal cual existen hoy.
