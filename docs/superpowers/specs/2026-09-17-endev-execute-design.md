# endev-execute — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-plan` (ver `2026-09-17-endev-plan-design.md`) — consume su
output (`.plans/YYYY-MM-DD-<tema>.md`).

## Objetivo

Un skill (`/endev-execute <ruta-al-plan>`) que corre **antes** de implementar
un plan: detecta qué tecnologías del plan no tienen una skill cubriéndolas,
resuelve eso (oficial si existe; si no, una nueva — global/comunidad,
solo del proyecto, o temporal/descartable según el caso), y depreca skills
propias que quedaron obsoletas porque ya salió una oficial — todo acotado a
lo que el plan en cuestión toca, nunca un barrido del repo entero.
`anti-cliche` corre como criterio transversal en cada paso, no como un paso
más. Al terminar, ejecuta el plan él mismo cuando hay paralelismo real
entre tareas (ver Flujo), o lo delega a `subagent-driven-development`/
`executing-plans` cuando es puramente secuencial.

## No-objetivos

- No es un linter de todo el repo `ai-agent-skills` — no audita skills que
  el plan actual no toca.
- No escribe a Sanity (sitio en producción) sin confirmación explícita del
  usuario en cada corrida — ver sección "Gate de Sanity".
- No implementa las tareas del plan — eso lo sigue haciendo
  `subagent-driven-development`/`executing-plans`, sin cambios respecto a
  cómo funcionan hoy.

## Reemplaza en `endev-plan`

El paso 8 (Handoff de ejecución) de `endev-plan` deja de ofrecer
`subagent-driven-development`/`executing-plans` directamente. En su lugar,
ofrece invocar `endev-execute <ruta-del-plan-recién-guardado>`, que hace la
auditoría de skills y luego dispara una de esas dos.

## Flujo

### 1. Escaneo de gaps

Lee el plan guardado (`.plans/...md`), extrae:

- La sección **Tech Stack** del header.
- Nombres de librerías/frameworks mencionados en rutas de archivo y bloques
  de código de las tareas.

Cruza esa lista contra los nombres de skills ya presentes en
`~/.claude/skills/` (coincidencia por nombre/alias, ej. "gsap" → skill
`gsap`, "framer-motion" → sin match). El resultado es una lista de
tecnologías **sin skill cubriéndolas**.

Si la lista queda vacía, se saltea el paso 2 (no hay nada que resolver)
pero el paso 3 (deprecación) igual corre — una skill ya cubierta puede
estar obsoleta aunque no sea un "gap".

### 2. Resolución por tecnología faltante

Primero, clasifica el gap:

- **Pública** — una librería/framework/herramienta con documentación
  pública propia (ej. gsap, framer-motion). Sigue el flujo de búsqueda de
  oficial de abajo.
- **Interna del proyecto** — una convención propia de ESE repo (ej. "cómo
  se estructuran los endpoints en `my-wallet-app`"), que no existe en
  ninguna doc pública porque no tiene sentido que exista. No hay "oficial"
  que buscar: se pasa directo a autoría, leyendo el propio código del
  proyecto en vez de documentación externa.
- **Temporal / puntual** — no es un patrón reusable, es una instrucción
  ad-hoc para resolver algo específico de esta corrida (ej. "cómo aplicar
  este refactor puntual"). No hay búsqueda de oficial ni pregunta de scope
  proyecto/global: se autora directo y se trata como descartable (ver
  "Skill temporal" más abajo).

Para gaps **públicos**, en este orden:

1. **Catálogo Anthropic + plugins ya instalados** — ¿alguno de los skills
   `anthropic-skills:*` o de un marketplace ya instalado (`claude-plugins-official`,
   etc.) cubre esto? Si sí, no hace falta nada más: se usa tal cual (siempre
   global, no aplica la pregunta de scope).
2. **Repo oficial del proyecto en GitHub** — ¿el propio repo de la
   librería/framework publica un `SKILL.md` oficial (raíz o
   `.claude/skills/`, `skills/`)? Si sí, se usa esa fuente: se copia (con
   atribución) o se referencia según su licencia (siempre global).
3. **Ninguna de las dos existe** → se autora una nueva desde la
   documentación oficial (WebFetch sobre su sitio de docs).

Para gaps **internos del proyecto**, se autora directamente leyendo el
código y las convenciones ya existentes en ese repo (sin búsqueda de
oficial — no aplica).

Para gaps **temporales**, se autora directo (sin búsqueda de oficial) y se
salta el resto de esta sección — va directo a "Skill temporal" más abajo,
no pasa por la pregunta de scope proyecto/global.

Para gaps **públicos** e **internos** (autoría persistente):

- Se redacta `SKILL.md` siguiendo el formato ya usado en `ai-agent-skills`
  (frontmatter `name`/`description`, secciones de uso, ejemplos).
- **Filtro `anti-cliche` antes de guardar nada:** ¿hace falta
  `reference.md`? Solo si la fuente es lo bastante extensa para
  justificarlo (precedente: `tailwind-css`, `shadcn-ui`, `gsap` sí lo
  tienen; skills chicas como `sitemap` no). ¿Hace falta `LICENSE.md`? Solo
  si el contenido no es 100% propio. Nunca se crean archivos "por las
  dudas".
- Metadata: `**Author**: Ender Puentes (con Claude Code)`, `**Source**:
  <URL de la documentación oficial usada, o "convención interna de
  <proyecto>" si es un gap interno>`.

**Dónde queda instalada (solo gaps públicos e internos) — se pregunta
siempre, sin heurística automática:**

> "La skill `<nombre>` es nueva. ¿La dejo solo en este proyecto
> (`<proyecto>/.claude/skills/<nombre>/`) o también la sumo a
> `ai-agent-skills` (global + candidata a tu sitio)?"

- **Solo proyecto:** se guarda en `<proyecto>/.claude/skills/<nombre>/`,
  se comitea al repo del proyecto (no a `ai-agent-skills`). No hay paso de
  Sanity ni de README del repo de skills — nunca sale de ese proyecto.
- **También global:** además de lo anterior (o en su reemplazo si no tiene
  sentido dentro del repo del proyecto, como en el caso de una librería
  pública), se guarda en `<repo-ai-agent-skills>/<nombre>/SKILL.md`, se
  symlinkea a `~/.claude/skills/<nombre>` (disponible de inmediato en esta
  misma ejecución), se comitea con Conventional Commits (`feat: agregar
  skill <nombre>`), y se prepara (sin ejecutar) el diff para sumarla a
  `endev/cms/.scripts/data/agent-skills.ts` y al `README.md` del repo de
  skills — ver "Gate de Sanity". Un gap interno del proyecto normalmente no
  se marca como global (no tiene sentido fuera de ese repo), pero la
  pregunta se hace igual por si el usuario decide lo contrario.

**Skill temporal (gaps puntuales, descartable):**

- Se redacta un `SKILL.md` mínimo (sin `reference.md` ni `LICENSE.md` —
  nunca se justifican para algo de un solo uso) en el directorio scratch de
  la sesión (`/tmp/claude-.../scratchpad/skills/<nombre>/`).
- Se symlinkea a `~/.claude/skills/<nombre>` para estar disponible durante
  esta corrida de `endev-execute` (incluyendo para los workers paralelos
  del paso 6 que la necesiten).
- No se comitea a ningún repo — ni al del proyecto ni a `ai-agent-skills`.
  No hay pregunta de scope ni paso de Sanity.
- Al terminar la ejecución del plan (éxito o abortada), se quita el
  symlink de `~/.claude/skills/` y se borra el archivo del scratchpad. No
  sobrevive a la sesión.

### 3. Barrido de deprecación (acotado al plan)

Para cada skill **propia** (no de terceros como `grill-me`) que el plan
actual usa: repetir la búsqueda de oficial del paso 2.1–2.2. Si ahora existe
una oficial que antes no existía:

- Mover `<repo>/<skill-name>/` a `<repo>/deprecated/<skill-name>/`.
- Agregar al `SKILL.md` movido una nota: `> Deprecada: reemplazada por
  <nombre/URL de la skill oficial>. Ver historial de este repo para el
  contenido original.`
- Quitar el symlink `~/.claude/skills/<skill-name>`.
- Comitear el movimiento (`chore: deprecar <nombre>, reemplazada por
  oficial`).
- Preparar (sin ejecutar) el cambio a `isActive: false` en el documento
  `resource-<skill-name>` de Sanity y la eliminación de la entrada en
  `agent-skills.ts` — ver "Gate de Sanity".

### 4. Gate de Sanity

Todo cambio a `agent-skills.ts` y todo write a Sanity (alta o baja) se
muestra como diff/resumen y **espera confirmación explícita** antes de:

- Ejecutar `seed-agent-skills.ts` (proyecto `84pkcrfz`, dataset
  `production`).
- O, para deprecaciones, correr el patch equivalente que setea
  `isActive: false` en el documento `resource-<skill-name>`.

El commit al repo `ai-agent-skills` en GitHub (paso 2 y 3) **no** requiere
esta confirmación — es un repo de herramientas propio, no el sitio público.

Si el usuario rechaza el write a Sanity, el cambio de repo queda igual
(commiteado); el desfase entre repo y sitio se menciona en el resumen final
como pendiente manual.

### 5. Análisis de paralelismo

Con las skills ya provistas, `endev-execute` (corriendo con `model: sonnet`
como orquestador) arma un grafo de dependencias a partir de la sección
**Interfaces** (`Consumes`/`Produces`) que cada tarea ya trae en el formato
de `writing-plans`:

- Dos tareas son candidatas a paralelo si ninguna consume lo que la otra
  produce, **y** no tocan los mismos archivos (columna `Files` de cada
  tarea). Si hay superposición de archivos, van secuenciales aunque no haya
  dependencia de interfaz — evita conflictos de merge.
- El grafo se agrupa en **olas**: cada ola es un conjunto de tareas
  ejecutables en simultáneo porque ya no dependen de nada pendiente.

Si todas las tareas terminan en una sola ola secuencial (sin ramas
independientes), no hay nada que paralelizar y se sigue el camino normal
del paso 6.

### 6. Handoff a ejecución

El resultado del paso 5 decide el camino:

- **Ninguna ola tiene más de una tarea (plan puramente secuencial):**
  `endev-execute` no aporta nada sobre lo ya probado — se ofrecen
  directamente las mismas dos opciones de `writing-plans`:
  `subagent-driven-development` (recomendado) o `executing-plans`.
- **Al menos una ola tiene más de una tarea (hay paralelismo real):**
  `endev-execute` orquesta la ejecución él mismo, ola por ola, con la
  mecánica de abajo. Las olas de una sola tarea dentro de ese mismo plan se
  ejecutan igual (un subagente, sin mensajería) — solo las olas
  multi-tarea usan el mecanismo paralelo completo.

**Mecánica de ejecución en olas (cuando hay paralelismo):**

- Se spawnea un subagente por tarea (`Agent`, `model: sonnet` para todos —
  sin heurística de "tarea simple → modelo barato", ver Decisiones), cada
  uno con **nombre** (`execute-<slug-de-la-tarea>`) para ser direccionable.
- Cada worker recibe **contexto acotado**: solo el bloque de su propia
  tarea del plan (Files/Interfaces/Steps), y solo el/los skill(s) que el
  paso 2 ya ubicó como relevantes para esa tarea — no el plan completo ni
  la lista completa de skills.
- El prompt de cada worker incluye los nombres de sus compañeros de ola y
  qué produce cada uno (columna `Produces` de sus Interfaces), con
  instrucción explícita: si necesita confirmar una firma o un detalle de
  interfaz, usar `SendMessage` al nombre del compañero en vez de asumirlo.
- Al terminar todos los workers de una ola, el orquestador revisa los
  outputs (mismo criterio de revisión entre tareas que
  `subagent-driven-development`) antes de spawnear la siguiente ola.

`anti-cliche` se aplica como criterio activo en todo momento durante esta
ejecución — código sin ambigüedad, sin archivos innecesarios, sin
sobre-abstracción — tanto en workers paralelos como secuenciales.

## Decisiones

- **Modelo de los workers paralelos: siempre Sonnet.** Se evaluó clasificar
  tareas como "mecánicas" (Haiku) vs "reales" (Sonnet) para ahorrar costo,
  pero se descartó: el riesgo de que la heurística subestime una tarea y
  entregue código de peor calidad pesa más que el ahorro. El orquestador
  también corre en Sonnet.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| No se puede determinar con certeza si algo es "oficial" | Se trata como no-oficial: se autora desde documentación (preferible tener una skill documentada a no tener nada) |
| La documentación oficial no es accesible (paywall, sin docs públicas) | No se crea la skill para esa tecnología; se avisa en el resumen final y se continúa con el resto del plan sin bloquear |
| El usuario rechaza el write a Sanity | Se deja constancia en el resumen; el repo de GitHub ya quedó actualizado igual |
| Una skill del plan ya está en `deprecated/` de una corrida anterior | No se reutiliza; se repite la búsqueda de oficial desde cero (pudo haber cambiado) |
| El plan no menciona ninguna tecnología sin cobertura | Se salta directo al handoff, sin mencionar el paso de auditoría |
| La ejecución del plan se aborta a mitad de camino (error, o el usuario corta) | Igual se limpia cualquier skill temporal creada — el cleanup no depende de que el plan haya terminado con éxito |

## Testing

Dry-run real: generar un plan de prueba con `endev-plan` que use una
librería sin skill (ej. `framer-motion`, que no está en el repo hoy) y
confirmar en la transcripción:

- Que detecta el gap correctamente.
- Que busca oficial en Anthropic/plugins y en el repo de GitHub de la
  librería antes de decidir escribir una propia.
- Que la skill nueva pasa por el filtro `anti-cliche` (no crea
  `reference.md` si no se justifica).
- Que pide confirmación antes de tocar Sanity, y que si se rechaza, el
  commit a GitHub ya se hizo igual.
- Repetir con un plan que use una skill ya deprecada previamente, para
  confirmar que se re-busca en vez de asumir el estado viejo.
- Generar un plan con dos tareas independientes (sin overlap de
  Interfaces ni de archivos) y confirmar que arma una ola con ambas, las
  corre en paralelo con nombres direccionables, y que al menos un worker
  usa `SendMessage` para confirmar un detalle con su compañero.
- Generar un plan con una tarea puntual (ej. un refactor de un archivo
  específico) y confirmar que la skill temporal se crea en el scratchpad,
  se usa, y se borra (symlink + archivo) al terminar la ejecución.

## Pendiente detectado (fuera de alcance de esta iteración)

`agent-skills.ts` tiene la entrada `anti-cliche-agent` pero la carpeta del
repo es `anti-cliche` — desalineación de slugs a corregir manualmente, no
bloquea este diseño.
