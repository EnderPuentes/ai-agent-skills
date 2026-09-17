# endev-execute — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)
**Depende de:** `endev-plan` (ver `2026-09-17-endev-plan-design.md`) — consume su
output (`.plans/YYYY-MM-DD-<tema>.md`).

## Objetivo

Un skill (`/endev-execute <ruta-al-plan>`) que corre **antes** de implementar
un plan: detecta qué tecnologías del plan no tienen una skill cubriéndolas,
resuelve eso (skill oficial si existe, o una nueva escrita desde la
documentación oficial si no), y depreca skills propias que quedaron
obsoletas porque ya salió una oficial — todo acotado a lo que el plan en
cuestión toca, nunca un barrido del repo entero. `anti-cliche` corre como
criterio transversal en cada paso, no como un paso más. Al terminar, dispara
la ejecución real vía `subagent-driven-development` o `executing-plans`.

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

Si la lista queda vacía, se salta directo al paso 5.

### 2. Resolución por tecnología faltante

Para cada tecnología de la lista, en este orden:

1. **Catálogo Anthropic + plugins ya instalados** — ¿alguno de los skills
   `anthropic-skills:*` o de un marketplace ya instalado (`claude-plugins-official`,
   etc.) cubre esto? Si sí, no hace falta nada más: se usa tal cual.
2. **Repo oficial del proyecto en GitHub** — ¿el propio repo de la
   librería/framework publica un `SKILL.md` oficial (raíz o
   `.claude/skills/`, `skills/`)? Si sí, se usa esa fuente: se copia (con
   atribución) o se referencia según su licencia.
3. **Ninguna de las dos existe** → se autora una nueva:
   - Se trae la documentación oficial del proyecto (WebFetch sobre su sitio
     de docs).
   - Se redacta `SKILL.md` siguiendo el formato ya usado en este repo
     (frontmatter `name`/`description`, secciones de uso, ejemplos).
   - **Filtro `anti-cliche` antes de guardar nada:** ¿hace falta
     `reference.md`? Solo si la documentación es lo bastante extensa para
     justificarlo (precedente: `tailwind-css`, `shadcn-ui`, `gsap` sí lo
     tienen; skills chicas como `sitemap` no). ¿Hace falta `LICENSE.md`?
     Solo si el contenido no es 100% propio (contenido derivado de otra
     fuente con licencia explícita). Nunca se crean archivos "por las
     dudas".
   - Metadata: `**Author**: Ender Puentes (con Claude Code)`,
     `**Source**: <URL de la documentación oficial usada>`.
   - Se guarda en `<repo>/<skill-name>/SKILL.md`, se symlinkea a
     `~/.claude/skills/<skill-name>` (disponible de inmediato en esta
     misma ejecución), se comitea al repo con Conventional Commits
     (`feat: agregar skill <nombre>`).
   - Se prepara (sin ejecutar) el diff para sumarla a
     `endev/cms/.scripts/data/agent-skills.ts` y al `README.md` del repo de
     skills — ver "Gate de Sanity".

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

### 5. Handoff a ejecución real

Con todas las skills necesarias ya provistas (oficiales, nuevas, o
deprecadas según corresponda), se ofrecen las mismas dos opciones que
`writing-plans`: `subagent-driven-development` (recomendado) o
`executing-plans`. `anti-cliche` se menciona explícitamente como criterio
activo durante toda la implementación que sigue — código sin ambigüedad,
sin archivos innecesarios, sin sobre-abstracción.

## Errores y casos borde

| Caso | Comportamiento |
|---|---|
| No se puede determinar con certeza si algo es "oficial" | Se trata como no-oficial: se autora desde documentación (preferible tener una skill documentada a no tener nada) |
| La documentación oficial no es accesible (paywall, sin docs públicas) | No se crea la skill para esa tecnología; se avisa en el resumen final y se continúa con el resto del plan sin bloquear |
| El usuario rechaza el write a Sanity | Se deja constancia en el resumen; el repo de GitHub ya quedó actualizado igual |
| Una skill del plan ya está en `deprecated/` de una corrida anterior | No se reutiliza; se repite la búsqueda de oficial desde cero (pudo haber cambiado) |
| El plan no menciona ninguna tecnología sin cobertura | Se salta directo al handoff, sin mencionar el paso de auditoría |

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

## Pendiente detectado (fuera de alcance de esta iteración)

`agent-skills.ts` tiene la entrada `anti-cliche-agent` pero la carpeta del
repo es `anti-cliche` — desalineación de slugs a corregir manualmente, no
bloquea este diseño.
