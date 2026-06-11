# Check: `single-responsibility`

## What to flag

- A **form** component/file that also owns unrelated concerns: routing, global query orchestration, unrelated modals, or feature-level state that could live in a **parent** or **hook** next to the page.
- A **list** view that embeds heavy **mutation** / **fetch** logic that is not about rendering the list (e.g. whole “create assistant” flow inside the list component).
- Pages that mix **many** unrelated domains without clear composition boundaries.

## Severity / confidence

- **medium**–**high** when the diff **adds** tight coupling (new imports of routers, mutations, or cross-feature stores into a presentational leaf).
- **low** when moving a few lines—might be acceptable; note “watch boundary.”

## False positives

- Small **local** UI state (open/close, field focus) inside a form—expected.
- Co-located **handlers** that only serve the form’s submit/validate flow.

## Guidance phrase for the audit

“Lift **orchestration** up; keep **this** component about **one** UI responsibility (form | list | card | layout).”
