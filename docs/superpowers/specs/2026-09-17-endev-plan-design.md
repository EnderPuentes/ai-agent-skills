# endev-plan — Diseño

**Fecha:** 2026-09-17
**Autor:** Ender (con Claude Code)

## Objetivo

Un skill de Claude Code (`/endev-plan`) que actúa como "cerebro" planificador
para cualquier repo: reúne contexto de código con el menor costo posible,
somete la idea a interrogatorio (`grill-me`), redacta un plan de
implementación con un modelo fuerte (Opus) y lo hace revisar
adversarialmente por un segundo modelo (Fable) antes de guardarlo. Reemplaza
el uso manual de `brainstorming` + `writing-plans` para el caso donde el
usuario quiere este tratamiento reforzado.

## No-objetivos

- No reemplaza `brainstorming`/`writing-plans` para tareas simples — sigue
  siendo válido invocarlos directo cuando no se necesita el circuito
  multi-modelo.
- No ejecuta el plan. Solo lo produce; la ejecución se delega a
  `subagent-driven-development` o `executing-plans`, igual que hace
  `writing-plans` hoy.
- No gestiona autenticación/créditos de `opencode` ni `ollama` — asume que ya
  están configurados en la máquina (verificado: `opencode` 1.18.31 con
  proveedor "OpenCode Zen"; `ollama` con `qwen3-coder:latest`).

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

### 2. Interrogatorio (`grill-me`)

Se invoca el skill `grill-me` con el contexto ya reunido, para resolver
ambigüedades del plan una pregunta a la vez con el usuario, igual que lo
haría `brainstorming` en su fase de clarificación — pero apoyado en lo que
ya se exploró del código, no solo en la descripción del usuario.

### 3. Borrador (Opus)

Un subagente (`Agent`, `model: opus`) redacta el plan completo siguiendo el
formato de `superpowers:writing-plans` (header, contexto, tareas
bite-sized, testing) usando el contexto del paso 1 y las respuestas del
paso 2.

### 4. Validación adversarial (Fable)

Un subagente (`Agent`, `model: fable`) recibe el borrador y lo audita
buscando huecos, riesgos no contemplados y supuestos débiles — no reescribe,
solo señala. Opus incorpora los ajustes válidos. Una sola ronda (sin loop).

**Manejo de fallo:** si la llamada a Fable falla, se reintenta una vez. Si
vuelve a fallar, el plan de Opus queda como final sin validación cruzada, y
el documento final incluye una nota explícita: "Validación Fable omitida
(fallo de API tras reintento)".

### 5. Estilo (`clinical-tone` + `anti-cliche`)

Se aplican ambos skills sobre la prosa final del documento (no sobre el
código de ejemplo que el plan pueda incluir) para eliminar relleno,
sicofantismo y clichés de redacción.

### 6. Seguridad condicional

Si el plan toca autenticación, secretos, RBAC, red o datos sensibles
(heurística: menciona alguno de esos términos o toca archivos como
`.env*`, `rbac.*`, middleware de auth), se corre el skill built-in
`security-review` sobre el plan antes de cerrarlo. Si no aplica, se omite
sin mencionarlo.

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
| `opencode` sin créditos/timeout | Cae a `ollama` (ver paso 1) |
| `ollama` no disponible | Cae a subagente Haiku (ver paso 1) |
| Los tres niveles de contexto fallan | Se avisa al usuario y se pregunta cómo seguir (no se asume contexto vacío) |
| Fable falla tras reintento | Plan queda solo con Opus, se anota en el documento |
| El repo actual no tiene `.plans/` | Se crea |
| El plan no toca nada sensible | Se omite `security-review` sin mencionarlo |

## Testing

Los skills son instrucciones, no código — no hay suite automatizada. La
validación es una corrida real de dry-run: invocar `/endev-plan` sobre una
tarea chica y real (por ejemplo, un cambio menor en `my-wallet-app`) y
confirmar en la transcripción:

- Que la cadena de fallback de contexto se anuncia correctamente.
- Que `grill-me` efectivamente pregunta antes de redactar.
- Que el borrador sale de un subagente Opus y la crítica de uno Fable.
- Que el documento final aparece en `.plans/` con el formato esperado.
- Forzar el fallo de Fable (ej. desconectando de internet) para confirmar
  el mensaje de fallback en el documento.

## Alcance de esta iteración

Un solo sub-proyecto: crear `endev-plan/SKILL.md` y enlazarlo. No incluye
cambios a `grill-me`, `clinical-tone`, `anti-cliche` ni a `writing-plans` —
se consumen tal cual existen hoy.
