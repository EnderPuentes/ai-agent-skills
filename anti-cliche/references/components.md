# React Component & Composition Clichés

## 1. Wrapper Components With No Logic

**The Cliché**: `<Wrapper>`, `<Box>`, `<Flex>`, `<Container>`, `<Section>` components that only apply a few Tailwind classes and forward all props. They fragment the component tree without adding meaning.

```tsx
// ❌
const Flex = ({ children, className, ...props }) => (
  <div className={cn('flex', className)} {...props}>{children}</div>
)

const PageWrapper = ({ children }) => (
  <main className="min-h-screen bg-gray-50 px-4 py-8">{children}</main>
)
// Used exactly once, contains no conditional logic
```

**The Fix**: Write the element directly at the call site. Wrappers earn their existence by encapsulating **logic**, not **layout**.

```tsx
// ✅
<main className="min-h-screen bg-gray-50 px-4 py-8">
  {children}
</main>
```

---

## 2. Single-Use Component Extraction

**The Cliché**: Extracting a sub-section of a component into its own file/component because it "feels big," even though it's only rendered once and has no reuse prospect.

**The Fix**: Colocate. A 150-line component is fine. Extract only when:
- The component is used in 2+ places, OR
- It manages its own independent state/effects, OR
- It has a meaningful, stable public API

---

## 3. Prop-Drilling Through Generic Wrappers

**The Cliché**: Passing `onClose`, `isOpen`, `variant`, `size` through 3–4 generic wrapper components that don't use them.

**The Fix**: Use React composition patterns. Pass rendered children, not data about how to render them.

```tsx
// ❌ Prop drilling
<Modal isOpen={open} onClose={close} title="Confirm" variant="danger" size="sm">
  <ConfirmBody text={message} onConfirm={handleConfirm} />
</Modal>

// ✅ Composition
<Modal isOpen={open} onClose={close}>
  <Modal.Header className="text-red-600">Confirm</Modal.Header>
  <Modal.Body>{message}</Modal.Body>
  <Modal.Footer>
    <Button onClick={handleConfirm}>Yes, delete</Button>
  </Modal.Footer>
</Modal>
```

---

## 4. `interface Props extends React.HTMLAttributes<HTMLDivElement>` by Default

**The Cliché**: Every component spreads `HTMLAttributes` and forwards a `ref` — even interactive wrappers that don't need to be composable at DOM level.

```tsx
// ❌ Applied reflexively to a one-off card
interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  title: string
}
const Card = React.forwardRef<HTMLDivElement, CardProps>(({ title, ...props }, ref) => (
  <div ref={ref} {...props}>{title}</div>
))
```

**The Fix**: Only extend HTML attributes and forward refs when the component is a **genuine primitive** meant for reuse across the design system. Application-level components have explicit, minimal prop interfaces.

---

## 5. Over-Memoization

**The Cliché**: Wrapping every component in `React.memo`, every value in `useMemo`, every callback in `useCallback` by default — before measuring any performance problem.

```tsx
// ❌
const value = useMemo(() => ({ id, name }), [id, name])  // trivial object
const handleClick = useCallback(() => setCount(c => c + 1), []) // stable setter
```

**The Fix**: Profile first. `useMemo` / `useCallback` add cognitive overhead. Use them when:
- A computation is genuinely expensive (>1ms measured)
- A reference-stable callback is required by a dependency array in a child that causes real re-renders

---

## 6. `useEffect` as the Default for Everything

**The Cliché**: Using `useEffect` to sync state that could be derived, or to fetch data that could be in a Server Component.

```tsx
// ❌
const [fullName, setFullName] = useState('')
useEffect(() => {
  setFullName(`${firstName} ${lastName}`)
}, [firstName, lastName])
```

**The Fix**: Derive, don't sync.

```tsx
// ✅
const fullName = `${firstName} ${lastName}`
```

---

## 7. Excessive `forwardRef` + Display Names

**The Cliché**: Every component gets `React.forwardRef` + `ComponentName.displayName = 'ComponentName'` as boilerplate, regardless of whether ref forwarding is needed.

**The Fix**: Add `forwardRef` only when a parent legitimately needs to access the DOM node (e.g., a trigger for a popover, focus management, animation). Most components don't.
