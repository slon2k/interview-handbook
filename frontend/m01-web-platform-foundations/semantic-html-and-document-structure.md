# Semantic HTML and Document Structure

## Definition

Semantic HTML uses elements according to their meaning and built-in behaviour. A heading describes document hierarchy, `nav` identifies navigation, a `button` performs an action, and an `a` element navigates to a URL. The browser exposes these meanings to assistive technology, search engines, and other tools.

## How It Works

- Use one logical heading hierarchy and landmarks such as `header`, `nav`, `main`, `aside`, and `footer` where they describe the page.
- Use lists for lists, tables for tabular relationships, and `figure`/`figcaption` when an illustration has a caption.
- Use a link for navigation and a button for an in-page action. Native controls provide keyboard behaviour and accessible semantics.
- Use `alt` text that conveys the purpose of an informative image. Use empty `alt` text for decorative images.
- Treat semantic structure as part of the component contract. A React component should expose the right HTML element, not just the right visual appearance.

## Alternatives & Trade-offs

Generic elements with ARIA can imitate some native semantics, but they do not automatically reproduce native keyboard behaviour, focus handling, or browser integration. A custom control is justified only when a native element cannot express the required interaction.

## Common Mistakes

- Using clickable `div` or `span` elements instead of buttons or links.
- Choosing elements by default browser styling rather than meaning, then overriding the styles.
- Using headings only to make text large or skipping heading levels without a structural reason.
- Using tables for page layout or lists for visual indentation alone.
- Giving an image redundant `alt` text that repeats an adjacent caption.

## Common Interview Questions

### Foundation

- What makes HTML semantic?
- When would you use a `button` instead of an `a` element?
- Why should a page have a logical heading structure?

### Intermediate

- How would you structure a dashboard containing navigation, filters, a results table, and a summary panel?
- What should the `alt` text be for a decorative icon, an informative chart, and a linked product image?

### Advanced and Follow-up

- Why can adding `role="button"` to a `div` still leave an inaccessible control?
- How would you review a React component that renders different visual variants but must preserve a meaningful underlying element?

## Practical Tasks

- Replace a `div`-based page with landmarks, headings, lists, links, buttons, and a correctly structured table.
- Review a React component API and identify where an `as` or `element` prop could accidentally allow invalid or inaccessible markup.

## Readiness Criteria

You can choose HTML elements by purpose, explain their native behaviour, structure a page for assistive technology, and identify semantic problems in React-rendered markup.

## References

- [MDN: HTML elements reference](https://developer.mozilla.org/docs/Web/HTML/Element)
- [MDN: HTML: A good basis for accessibility](https://developer.mozilla.org/docs/Learn/Accessibility/HTML)
