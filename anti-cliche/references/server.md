# Server Components & Server Actions Clichés — Next.js App Router

## 1. `"use client"` by Default

**The Cliché**: Adding `"use client"` to every component file as a reflex — because "it's safer" or "we might need hooks later."

**Why it's wrong**: You opt out of RSC benefits (no client JS bundle, direct DB access, no hydration cost) for zero gain. The whole point of the App Router is that components are Server by default.

**The Fix**: Only add `"use client"` when the component uses:
- Browser APIs (`window`, `document`, `navigator`)
- Event handlers that require interactivity (onClick, onChange on controlled inputs)
- React hooks that require client state (`useState`, `useEffect`, `useRef`, `useContext` with client providers)
- Third-party components that themselves require client (e.g. a chart library)

```tsx
// ❌ — no hooks, no events, no browser APIs
'use client'
export default function StaticCard({ title, body }: { title: string; body: string }) {
  return <div><h2>{title}</h2><p>{body}</p></div>
}

// ✅ — just a Server Component
export default function StaticCard({ title, body }: { title: string; body: string }) {
  return <div><h2>{title}</h2><p>{body}</p></div>
}
```

---

## 2. Fetching Data in Client Components When Server Components Are Available

**The Cliché**: Using `useEffect` + `fetch` or `SWR` / `React Query` to load data that could be fetched synchronously in a Server Component.

```tsx
// ❌
'use client'
export default function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers)
  }, [])
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
}
```

**The Fix**: Fetch directly in the Server Component. No loading state, no waterfall, no client JS.

```tsx
// ✅
export default async function UserList() {
  const users = await db.user.findMany()
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
}
```

Use SWR/React Query only when you need:
- Client-side polling or real-time updates
- Optimistic UI with client-side mutation
- Data that changes based on client interaction after initial load

---

## 3. Server Actions That Just Delegate

**The Cliché**: A Server Action that does nothing but call a function that could be the body of the action itself.

```ts
// ❌
// actions/user.ts
'use server'
import { updateUser } from '@/services/userService'

export async function updateUserAction(id: string, data: UserData) {
  return updateUser(id, data)  // userService.updateUser is just db.user.update
}
```

**The Fix**: If the service function has no other consumers, inline it.

```ts
// ✅
'use server'
export async function updateUser(id: string, data: UserData) {
  return db.user.update({ where: { id }, data })
}
```

---

## 4. Silent Error Swallowing in Server Actions

**The Cliché**: Wrapping every Server Action body in `try/catch` that logs to console and returns `{ error: 'Something went wrong' }` — hiding the real error type and losing the stack.

```ts
// ❌
export async function createPost(data: PostData) {
  try {
    return await db.post.create({ data })
  } catch (e) {
    console.error(e)
    return { error: 'Something went wrong' }
  }
}
```

**The Fix**: Let errors propagate to the error boundary, or catch *specific* known error types with *specific* messages.

```ts
// ✅ — handle only what you can meaningfully recover from
export async function createPost(data: PostData) {
  try {
    return await db.post.create({ data })
  } catch (e) {
    if (e instanceof Prisma.PrismaClientKnownRequestError && e.code === 'P2002') {
      return { error: 'A post with this slug already exists.' }
    }
    throw e // let unknown errors propagate
  }
}
```

---

## 5. Route Handler (`route.ts`) for Everything

**The Cliché**: Creating `app/api/*/route.ts` endpoints for data mutations that could be Server Actions, just because API routes feel "familiar."

**When to use Route Handlers**:
- Webhooks from third-party services
- Responses that aren't JSON (file downloads, streams, redirects for OAuth)
- Endpoints consumed by external clients or mobile apps

**When to use Server Actions instead**:
- Form submissions
- Button-triggered mutations
- Any mutation triggered only by your own Next.js UI

---

## 6. Suspense Boundary Inflation

**The Cliché**: Wrapping every async Server Component in its own `<Suspense>` with a unique skeleton, creating a janky waterfall of independently loading UI chunks.

**The Fix**: Group related async work under a single `<Suspense>` boundary that matches a meaningful UX unit. A page's main content should load as a unit, not as 7 independent skeleton boxes.

---

## 7. `revalidatePath('/')` as a Shortcut

**The Cliché**: Revalidating the entire root path after every mutation because it's easier than thinking about which paths actually changed.

```ts
// ❌
revalidatePath('/')  // nukes the entire cache every time a comment is posted
```

**The Fix**: Revalidate the specific path or tag.

```ts
// ✅
revalidatePath(`/posts/${postId}`)
// or with tags:
revalidateTag(`post-${postId}`)
```
