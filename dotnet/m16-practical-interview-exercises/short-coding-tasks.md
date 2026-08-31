# Short Coding Tasks

## What This Assesses

A short coding task (typically solvable in 5-15 minutes, often live or on a shared screen) checks whether you can translate a plain-language requirement into working code under mild time pressure — not algorithmic brilliance, but clean, correct, idiomatic C#. Interviewers are watching *how* you approach the problem (do you clarify ambiguous requirements first?) as much as the final code.

## Format and Time Expectations

Usually presented verbally or as a short written prompt, solved in a shared editor or on a whiteboard/doc. Expect no test framework or IDE tooling (no IntelliSense, sometimes no compiler) — correctness by careful reading, not by trial and error.

## Exercise 1: FizzBuzz Variant

**Problem:** Write a method that, given a list of integers, returns a list of strings where each number divisible by 3 becomes "Fizz", divisible by 5 becomes "Buzz", divisible by both becomes "FizzBuzz", and otherwise the number itself as a string.

**What a strong answer demonstrates:** Checking divisible-by-both *before* the individual checks (or checking both conditions in one combined branch) rather than letting the by-3 check fire first and never reaching the FizzBuzz case; using `string` conversion correctly; clean control flow (a `switch` expression or ordered `if`s), not deeply nested conditionals.

**Common mistakes:** Checking divisible-by-3 and divisible-by-5 as separate top-level `if`s without an `else`, causing a number divisible by both to append "Fizz" then separately "Buzz" instead of "FizzBuzz" — or never reaching the FizzBuzz case at all depending on the exact structure.

## Exercise 2: Balanced Parentheses

**Problem:** Given a string containing `(`, `)`, `[`, `]`, `{`, `}`, determine whether the brackets are balanced and correctly nested.

**What a strong answer demonstrates:** Recognizing this as a stack problem (push opening brackets, pop and match on closing brackets) — a classic pattern worth recognizing instantly rather than reasoning about from scratch. Correct handling of the edge cases: an empty string (balanced, trivially), a string with only closing brackets (unbalanced), a closing bracket that doesn't match the most recent opening one.

**Common mistakes:** Just counting total opening vs. closing brackets (which passes "([)]" incorrectly, since the *count* matches but the *nesting* doesn't) instead of actually tracking order via a stack.

## Exercise 3: Reverse Words in a Sentence, In Place Conceptually

**Problem:** Given a sentence, return it with the order of words reversed but the words themselves unreversed — `"the quick fox"` becomes `"fox quick the"`.

**What a strong answer demonstrates:** Correct handling of multiple consecutive spaces or leading/trailing whitespace (should the output normalize spacing, or preserve it exactly? — worth clarifying with the interviewer, which itself is a positive signal). A clean split/reverse/join, or a from-scratch two-pointer approach if asked to avoid built-in split/join methods.

**Common mistakes:** Reversing the entire string character-by-character instead of reversing word order specifically (`"the quick fox"` incorrectly becomes `"xof kciuq eht"`), confusing the two very different problems.

## Exercise 4: First Non-Repeating Character

**Problem:** Given a string, return the first character that appears exactly once, or indicate if none exists.

**What a strong answer demonstrates:** Using a frequency map (`Dictionary<char, int>`, Module 3) built in one pass, then a second pass over the original string (not the dictionary, which has no guaranteed order) to find the first character with count 1 — correctly preserving original order.

**Common mistakes:** Iterating over the dictionary to find the answer instead of iterating over the original string a second time, risking wrong ordering since dictionary enumeration order isn't guaranteed to match insertion order.

## Readiness Criteria

Solve each of these within roughly 10 minutes without IDE assistance, correctly handling the edge cases named above, and verbally narrate your approach and any ambiguous-requirement clarifications as you go rather than solving in silence.

## References

- [Selecting the correct collection (Module 3)](../m03-collections-linq/selecting-collections.md)
- [Dictionaries (Module 3)](../m03-collections-linq/dictionaries.md)
