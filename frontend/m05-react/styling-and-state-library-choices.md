# Styling and State-Library Choices

## Definition

React doesn't mandate a styling approach or a state-management library — both are architectural choices layered on top. Styling options range from plain CSS to CSS Modules (build-time scoped class names), utility-first frameworks (Tailwind), and CSS-in-JS. Client-state libraries (Redux, Zustand, Jotai) exist specifically for *shared client state* — they solve a different problem than server state (previous topic), which they should not be used to reimplement.

```tsx
import styles from "./Card.module.css"; // CSS Modules: class names are scoped, generated uniquely per file at build time
<div className={styles.card}>...</div>
```

## Alternatives & Trade-offs

Plain global CSS is the simplest mental model but offers no protection against class-name collisions across a large codebase — exactly the specificity/cascade risk Module 1 covers, now at application scale. CSS Modules solve the collision problem by scoping class names automatically at build time, at the cost of an extra build step and slightly more indirection between a class name in markup and its actual generated name. Utility-first CSS (Tailwind) trades writing custom class names for composing pre-defined ones directly in markup, which some teams find faster to work in and others find noisy. CSS-in-JS colocates styles with components, at the cost of runtime or build-tooling overhead that varies significantly by library.

## How It Works

### CSS Modules — scoped class names without changing how CSS itself is written

```css
/* Card.module.css */
.card { padding: 16px; border-radius: 8px; }
```

```tsx
import styles from "./Card.module.css";

function Card({ children }: { children: ReactNode }) {
  return <div className={styles.card}>{children}</div>;
  // styles.card resolves to something like "Card_card__a1b2c" at build time —
  // a DIFFERENT file's .card class can never accidentally collide with this one
}
```

### Utility-first CSS — composing pre-defined classes directly in markup

```tsx
function Card({ children }: { children: ReactNode }) {
  return <div className="p-4 rounded-lg shadow-sm bg-white">{children}</div>;
  // no custom CSS file at all — every visual property is expressed via existing utility classes
}
```

This trades "naming things" for "composing from a fixed vocabulary" — faster for many teams once the utility vocabulary is familiar, but the markup itself carries more visual detail than a single semantic class name would.

### Choosing a client-state library — only after ruling out simpler ownership

```tsx
// Zustand: a lightweight store, read via a hook, with no Provider wrapping required
const useCartStore = create<{ items: Item[]; addItem: (item: Item) => void }>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));

function CartBadge() {
  const itemCount = useCartStore((state) => state.items.length); // only rerenders when items.length actually changes
  return <span>{itemCount}</span>;
}
```

The state-categories topic's decision process still applies before reaching for this: is this value actually shared client state (not server data, not something that should live in the URL, not something local to one small subtree)? Only once that's confirmed does choosing *which* library (Redux's explicit action/reducer model vs. Zustand's lighter direct-mutation-style API) become the relevant question.

### The mistake this topic exists to prevent — treating server data as client state

```tsx
// WRONG: manually copying fetched data into a client store, then reimplementing caching/invalidation by hand
const useOrdersStore = create<{ orders: Order[]; setOrders: (o: Order[]) => void }>((set) => ({
  orders: [],
  setOrders: (orders) => set({ orders }),
}));
// every consumer now has to manually figure out when this cached copy is stale and needs refetching —
// exactly the problem the previous topic's server-state library already solves
```

## Application

Choose a styling approach based on team conventions, build-tooling fit, and how the team prefers to express visual detail (semantic class names vs. utility composition) — accessibility fundamentals from Module 1 apply regardless of which is chosen. Choose a client-state library only after confirming, via the state-categories decision process, that the value is genuinely shared client state — never as a place to cache server data, which belongs in a server-state library instead.

## Common Mistakes

- Choosing a styling approach by popularity or personal preference without considering the team's existing build pipeline or design-system integration.
- Using unscoped global class names that collide across unrelated features as an application grows.
- Copying fetched server data into a client-state store "to make it globally available," then manually reimplementing caching and invalidation that a server-state library already provides.
- Reaching for a global store to avoid lifting a small, genuinely local piece of state one level up the component tree.

## Common Interview Questions

### Basic
- What problem do CSS Modules solve compared to plain global CSS?
- What's the difference between a client-state library and a server-state library?

### Intermediate
- What are the trade-offs between CSS Modules and a utility-first approach like Tailwind?
- Why shouldn't server-fetched data typically be copied into a client-state store?

### Advanced
- How would you choose a styling strategy for a multi-team application sharing one design system?
- How would you decide, for a specific value, whether it belongs in local state, a client-state store, or a server-state library?

### Follow-up Questions
- Does using CSS Modules require giving up ordinary CSS features like the cascade or media queries?
- Is a client-state library ever an appropriate place to cache data fetched from an API?

### Code Prediction
Given a Zustand store holding both a UI-only value (a sidebar's open/closed state) and a copy of API-fetched order data manually kept "in sync," what maintenance burden does the second value create that the first one doesn't?

## Practical Tasks

- Compare CSS Modules and a utility-first approach for the same small component, and justify a choice based on maintainability and team fit.
- Identify server-fetched data incorrectly stored in a client-state store and refactor it to use a server-state library instead.
- Design a Zustand or equivalent store for a genuinely client-owned value (like a multi-step wizard's current step), justifying why it doesn't belong in local component state.

## Readiness Criteria

Compare styling approaches by maintainability and team fit rather than popularity, and correctly distinguish genuine shared client state from server data that should never be duplicated into a client store.

## References

- [React: Thinking in React](https://react.dev/learn/thinking-in-react)
- [Zustand documentation](https://zustand.docs.pmnd.rs/)
- [Redux documentation](https://redux.js.org/)
