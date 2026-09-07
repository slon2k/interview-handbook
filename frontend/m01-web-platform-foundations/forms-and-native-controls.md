# Forms and Native Controls

## Definition

HTML forms and their native controls (`<input>`, `<select>`, `<textarea>`, `<button>`) come with built-in behavior — label association, keyboard navigation, basic validation, and submission handling — that a hand-built equivalent has to reimplement, usually incompletely. Choosing the right native control and wiring it correctly is the single highest-leverage accessibility decision most forms make.

```html
<form>
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required>
  <button type="submit">Sign up</button>
</form>
```

## Alternatives & Trade-offs

Building a custom control (a styled `<div>`-based dropdown, a custom checkbox) gives full visual control beyond what native controls easily allow, but requires manually reimplementing keyboard interaction, focus management, and accessible naming — all of which a native control provides automatically. Native controls are harder to restyle in some browsers but are the correct default whenever the native behavior is acceptable, which is most of the time.

## How It Works

### `<label for>` — the single most important form-accessibility wire-up

```html
<!-- WRONG: visually near the input, but not programmatically associated with it -->
<span>Email</span>
<input type="email" id="email">

<!-- RIGHT: clicking the label focuses the input, and screen readers announce the label with the input -->
<label for="email">Email</label>
<input type="email" id="email">

<!-- Also correct: wrapping the input inside the label avoids needing a matching id/for pair -->
<label>
  Email
  <input type="email" name="email">
</label>
```

Without this association, clicking the visible text does nothing, and a screen reader announces the input with no name at all — a "spans near an input" pattern looks identical sighted but is functionally broken for anyone using assistive technology.

### Choosing the right `<input type>` — free validation and the right mobile keyboard

```html
<input type="email">   <!-- browser validates email shape; mobile keyboards show @ and .com shortcuts -->
<input type="tel">      <!-- numeric-friendly keyboard on mobile, no built-in format validation -->
<input type="number">    <!-- numeric keyboard AND built-in min/max/step validation -->
<input type="text">       <!-- the generic fallback — none of the above benefits -->
```

Using `type="text"` for an email field throws away free validation and the correct mobile keyboard for no benefit — the specific type should match the actual data being collected.

### Native validation, and when to layer custom validation on top

```html
<input type="email" required minlength="5">
```

```javascript
const input = document.querySelector('input');
if (!input.checkValidity()) {
  console.log(input.validationMessage); // the browser's own, localized error message
}
```

Native attributes (`required`, `minlength`, `pattern`) cover a large fraction of real validation needs with zero JavaScript, and their error messages are already localized to the user's browser language — reimplementing all of this in JavaScript from scratch is extra work that's easy to get less complete than the native version.

### Associating an error message with its field for assistive technology

```html
<label for="email">Email</label>
<input type="email" id="email" aria-describedby="email-error" aria-invalid="true">
<span id="email-error">Enter a valid email address</span>
```

`aria-describedby` links the error text to the input so a screen reader announces it alongside the field, not just as an isolated, unconnected message somewhere else on the page.

### Submit behavior — letting the browser do the work

```html
<form>
  <input type="text" name="query">
  <button type="submit">Search</button> <!-- Enter in the input ALSO submits the form, for free -->
</form>
```

A `<button type="submit">` inside a `<form>` gets both click and Enter-key submission automatically — a `<div>`-based "submit button" needs a manual keydown handler to replicate the Enter-key behavior most users expect by default.

## Application

Use the most specific native `<input type>` for the data being collected, always associate labels via `for`/`id` or wrapping, rely on native validation attributes before reaching for custom JavaScript validation, and link error messages to their field with `aria-describedby`.

## Common Mistakes

- Using a `<span>` or plain text near an input instead of a properly associated `<label>`.
- Using `type="text"` for email, phone, or numeric fields, losing free validation and the correct mobile keyboard.
- Building a custom dropdown or checkbox from `<div>`s without reimplementing the keyboard interaction a native `<select>` or `<input type="checkbox">` provides automatically.
- Displaying a validation error visually without connecting it to the field via `aria-describedby`, leaving screen reader users unaware it exists.
- Using a `<div>` with a click handler instead of `<button type="submit">`, losing automatic Enter-key submission.

## Common Interview Questions

### Basic
- Why does `<label for="...">` matter, beyond visual placement near the input?
- What's the benefit of using `type="email"` instead of `type="text"`?

### Intermediate
- What does `aria-describedby` do, and when would you use it on a form field?
- What native validation attributes can replace simple custom JavaScript validation?

### Advanced
- What would you need to rebuild manually if you replaced a native `<select>` with a custom `<div>`-based dropdown?
- How would you design error-message association so both sighted and screen-reader users clearly connect an error to its field?

### Follow-up Questions
- Does `required` alone provide a fully accessible validation experience?
- Can a `<label>` wrap its input instead of using a separate `for`/`id` pair?

### Code Prediction
Given a `<span>Email</span>` placed visually next to an `<input>` with no `for`/`id` association, what happens when a sighted mouse user clicks the word "Email"? What does a screen reader announce when it reaches that input?

## Practical Tasks

- Fix a form where labels are visually placed but not programmatically associated with their inputs.
- Replace `type="text"` fields with the correct specific input types for an email, phone, and quantity field.
- Add `aria-describedby`-linked error messages to a form with existing but disconnected error text.

## Readiness Criteria

Correctly associate labels with controls, choose specific input types deliberately, rely on native validation where sufficient, and connect error messages to their fields for assistive technology.

## References

- [MDN: Your first form](https://developer.mozilla.org/docs/Learn/Forms/Your_first_form)
- [MDN: input element](https://developer.mozilla.org/docs/Web/HTML/Element/input)
- [MDN: Form data validation](https://developer.mozilla.org/docs/Learn/Forms/Form_validation)
