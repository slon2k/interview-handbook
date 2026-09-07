# Semantic HTML and Document Structure

## Definition

Semantic HTML means choosing elements for what they *mean*, not how they happen to look by default — a `<button>` because it's a button, not a `<div>` styled to resemble one. The browser, assistive technology, and search engines all rely on this meaning to provide behavior (keyboard support, focus handling) and structure (a navigable outline) that a generic element never gets for free.

```html
<!-- Non-semantic: LOOKS like a page, but means nothing to a screen reader or a browser -->
<div class="header">
  <div class="nav">
    <div class="link">Home</div>
  </div>
</div>
<div class="main">
  <div class="title">Article Title</div>
</div>

<!-- Semantic: the SAME visual result, but the structure carries real meaning -->
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <h1>Article Title</h1>
</main>
```

## Alternatives & Trade-offs

A `<div>`-only approach gives complete visual control with no default browser behavior to work around, but every piece of functionality a native element would have provided for free — keyboard activation, an accessible role, a document outline — has to be rebuilt by hand, usually incompletely. Semantic elements trade a small amount of default styling you'll often override anyway for behavior and structure that would otherwise require a lot of ARIA and JavaScript to approximate, and even then rarely as robustly.

## How It Works

### Landmark elements build a navigable page outline

```html
<body>
  <header>...</header>
  <nav>...</nav>
  <main>
    <article>
      <h1>Post Title</h1>
      <section>
        <h2>Section Heading</h2>
      </section>
    </article>
    <aside>Related links</aside>
  </main>
  <footer>...</footer>
</body>
```

Screen reader users commonly navigate by landmark or by heading level, jumping directly to `<main>` or between `<h1>`/`<h2>` elements — a page built entirely from `<div>`s has no such outline for them to jump through at all, regardless of how it looks visually.

### Heading levels must be sequential to form a real outline

```html
<!-- WRONG: skips from h1 to h3, breaking the outline a screen reader user navigates by -->
<h1>Page Title</h1>
<h3>Subsection</h3>

<!-- RIGHT: sequential levels, chosen for structure, not for font size -->
<h1>Page Title</h1>
<h2>Subsection</h2>
```

Heading levels should never be chosen because "this needs to look smaller" — that's what CSS is for. `<h3>` used only because it renders at a convenient font size, while skipping `<h2>`, breaks the actual document outline for anyone navigating by heading.

### `<button>` vs. a styled `<div>` — behavior you get for free

```html
<!-- A real button: Enter AND Space both activate it, it's focusable, it has an accessible role automatically -->
<button type="button" onclick="handleClick()">Submit</button>

<!-- A div "styled as a button": none of that works without extra code -->
<div class="button-like" onclick="handleClick()">Submit</div>
```

The `<div>` version isn't focusable by default, doesn't respond to Enter or Space, and has no accessible role — all three would need to be added manually (`tabindex`, keydown handlers, `role="button"`) to approximate what `<button>` already does, and it's easy to miss one of the three.

### `<article>` vs. `<section>` vs. `<div>` — a genuine, testable distinction

```html
<article>   <!-- independently distributable/reusable content: a blog post, a product card -->
<section>   <!-- a thematic grouping WITHIN a page, usually with its own heading -->
<div>        <!-- no semantic meaning at all — a pure styling/grouping hook -->
```

A useful test: if the content would still make sense syndicated on its own (an RSS feed entry, a card in a feed), it's an `<article>`. If it's just one thematic part of the current page, it's a `<section>`. If it carries no meaning of its own, it's a `<div>`.

## Application

Choose the most specific element that matches the actual meaning before reaching for a generic `<div>`/`<span>` plus ARIA. Use landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`) once per page (except `<nav>`, which can repeat), and keep heading levels sequential to preserve a real document outline.

## Common Mistakes

- Choosing an element based on its default appearance rather than its meaning, then fighting the CSS to make it look different.
- Skipping heading levels to get a particular default font size, breaking the document outline.
- Using `<div onclick="...">` instead of `<button>`, silently losing keyboard activation and the accessible role.
- Using more than one `<main>` element per page, or nesting one `<main>` inside another.
- Reaching for ARIA roles to re-describe an element when a native element with that role built in was available all along.

## Common Interview Questions

### Basic
- What does "semantic HTML" mean?
- What's the difference between `<article>` and `<section>`?

### Intermediate
- What specifically does a `<button>` give you for free that a styled `<div>` doesn't?
- Why should heading levels stay sequential even if that means overriding the default font size with CSS?

### Advanced
- How does a screen reader user typically navigate a well-structured page, and what does that imply about landmark and heading usage?
- How would you decide whether a given block of content should be an `<article>` or a `<section>`?

### Follow-up Questions
- Can a page have more than one `<nav>` element?
- Does using semantic elements guarantee an accessible page on its own?

### Code Prediction
Given the `<div onclick="handleClick()">Submit</div>` example above, can a keyboard-only user (no mouse) activate this element by pressing Enter or Space, without any additional JavaScript? What would need to be added to make it behave like a real button?

## Practical Tasks

- Convert a `<div>`-only page layout into one using landmark elements, correct heading levels, and a real `<button>` for its interactive control.
- Identify a heading-level skip in a sample page and fix it without changing the visual design.
- Decide, for a list of content blocks (a blog post, a sidebar widget, a page section), whether each should be an `<article>`, `<section>`, or `<div>`.

## Readiness Criteria

Choose elements by meaning rather than default appearance, keep heading levels sequential, and explain concretely what behavior a native element provides that an equivalent styled `<div>` does not.

## References

- [MDN: HTML elements reference](https://developer.mozilla.org/docs/Web/HTML/Element)
- [MDN: Document and website structure](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Document_and_website_structure)
