# UI/UX Clichés

## 1. The Generic "AI/Startup" Hero

**The Cliché**: Massive centered hero with:
- Gradient text on the headline (`bg-gradient-to-r from-blue-500 to-purple-600 bg-clip-text text-transparent`)
- Floating 3D blobs or geometric shapes as background decoration
- Glassmorphism cards (`backdrop-blur-md bg-white/10 border border-white/20`) everywhere
- "Trusted by X+ companies" logos with no context

**The Fix**: Design with specific intent. If your product is a data dashboard, the hero should look like a dashboard preview. If it's a writing tool, show text being written. Decoration that doesn't communicate domain-specific value is noise.

---

## 2. Aesthetics Breaking Usability

**The Cliché**:
- `text-gray-400` on `bg-white` for body text because it "looks clean" (fails WCAG AA)
- Hiding navigation in a hamburger menu on desktop to save space
- Buttons styled as plain text links, or links styled as buttons
- 8px font sizes labeled as "captions" that users need to squint to read

**The Fix**: Accessibility before aesthetics. Minimum contrast ratio: 4.5:1 for body text (WCAG AA). Navigation visible at the viewport size where it matters. Interactive affordances must be unambiguous.

---

## 3. Scroll-Triggered Animation on Everything

**The Cliché**: Every section, card, and paragraph has a `framer-motion` `whileInView` animation. The page feels like a PowerPoint presentation.

```tsx
// ❌
<motion.div
  initial={{ opacity: 0, y: 20 }}
  whileInView={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.5 }}
>
  <p>Our pricing is simple.</p>
</motion.div>
```

**The Fix**: Animate to communicate state changes (loading → loaded, error → resolved) or to guide attention to a single primary action. Static content doesn't need to announce itself.

---

## 4. Skeleton Loaders That Don't Match Content

**The Cliché**: A generic `<Skeleton className="h-4 w-full" />` row that bears no resemblance to the actual card, table, or list it's placeholding for.

**The Fix**: Skeletons should structurally mirror what they replace. Users should be able to predict the incoming content from the skeleton shape. If building accurate skeletons is too expensive, use a spinner or a simple "Loading…" text instead of a misleading skeleton.

---

## 5. Empty State as an Afterthought

**The Cliché**: Empty states that render nothing, or a bare `<p>No items found.</p>` with no action, no explanation, and no visual.

**The Fix**: Every empty state answers three questions:
1. Why is this empty? (no data yet, filtered to zero, error)
2. What can the user do about it?
3. Where should they go next?

---

## 6. Modals for Everything

**The Cliché**: Reaching for a modal dialog for every non-trivial interaction: confirmation, editing a field, displaying detail, showing an image.

**When a modal is appropriate**:
- Confirming a destructive or irreversible action
- A short form (≤4 fields) that saves without leaving the current context

**When to use something else**:
- Detail views → navigate to a detail page or use a slide-over panel
- Inline editing → inline edit in place
- Alerts and notifications → toast, banner, or inline message

---

## 7. The Dashboard Grid of KPI Cards

**The Cliché**: 4 identical `<StatCard>` components in a grid showing "Total Users", "Revenue", "Active Sessions", "Churn Rate" — each with an icon, a big number, and a percentage delta — regardless of what the user actually needs to act on.

**The Fix**: Display data that drives decisions, not data that fills space. If the user can't act differently based on seeing a number, question whether that number should be on the primary screen.

---

## 8. Toast for Everything

**The Cliché**: Showing a success toast after every mutation, including ones where the UI itself already reflects the change.

```
// ❌ User clicks "Mark as complete" → checkbox checks → toast appears saying "Marked as complete!"
```

**The Fix**: Use toasts for async outcomes that aren't visually obvious (background jobs, email sent, export ready). Don't narrate what the user just watched happen.
