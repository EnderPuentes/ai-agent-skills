# Architecture Clichés — Next.js / TypeScript

## 1. Overengineering (YAGNI Violations)

**The Cliché**: Solving a simple data-fetch with a service layer, repository pattern, and dependency injection container because "it might scale."

**Banned structures for simple Next.js apps**:
```ts
// ❌ AbstractUserRepository → UserRepositoryImpl → UserService → UserFacade
// ❌ Generic<T extends Entity> base classes for 2 models
// ❌ EventBus for a form submission
// ❌ DI containers (InversifyJS, tsyringe) in a Next.js app route
```

**The Fix**: Write the fetch directly where it's consumed. Extract only when you have 3+ actual duplications.

```ts
// ✅ In the Server Component
const user = await db.user.findUnique({ where: { id: params.id } })
```

---

## 2. Barrel File Abuse

**The Cliché**: Every folder has `index.ts` re-exporting everything so imports look "clean". This breaks tree-shaking, hides real module structure, and creates circular dependency risk.

```ts
// ❌ components/index.ts
export * from './Button'
export * from './Card'
export * from './Modal'
// … 40 more lines
```

**The Fix**: Import directly from the file. The path is not noise — it's information.

```ts
// ✅
import { Button } from '@/components/Button'
import { Card } from '@/components/Card'
```

---

## 3. God Utility Files

**The Cliché**: `utils/helpers.ts` or `lib/utils.ts` as a catch-all for unrelated functions that grow forever.

**The Fix**: Name by domain. If it formats dates, it's `lib/format-date.ts`. If it validates emails, it's `lib/validate-email.ts`. No file named `helpers`, `utils`, `misc`, or `common` unless it specifically houses one cohesive concern.

---

## 4. Breaking Existing Patterns

**The Cliché**: The repo uses SWR for client fetching — a new developer introduces React Query "because it's better." The repo uses Prisma — a new file introduces raw SQL via a different driver.

**The Fix**: Respect the existing domain model. Consistency > novelty. If the existing pattern is genuinely broken, refactor it wholesale with team agreement — don't introduce a parallel system.

---

## 5. Type Inflation

**The Cliché**: Creating a TypeScript `interface` or `type` for every tiny object shape, even inline one-offs.

```ts
// ❌
interface ButtonClickHandlerEventPayload {
  event: React.MouseEvent<HTMLButtonElement>
}
const handleClick = (payload: ButtonClickHandlerEventPayload) => { ... }
```

**The Fix**: Inline types where they're only used once. Extract only when the type is shared or complex.

```ts
// ✅
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => { ... }
```

---

## 6. Premature Abstraction via `lib/` Layering

**The Cliché**: Every feature gets a `lib/feature/client.ts`, `lib/feature/server.ts`, `lib/feature/types.ts`, `lib/feature/constants.ts`, `lib/feature/utils.ts` — before a single real requirement justifies the split.

**The Fix**: Start with one file. Split only when a file exceeds ~200 lines *and* has genuinely separable concerns with real reuse.
