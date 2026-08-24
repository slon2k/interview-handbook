# Arrays

## Definition

An array stores a fixed number of elements of one type in a contiguous, zero-based sequence.

```csharp
int[] values = [10, 20, 30];
Console.WriteLine(values[1]);
```

Multidimensional arrays use rectangular dimensions; jagged arrays contain arrays of potentially different lengths.

## Alternatives & Trade-offs

Arrays provide fast indexed access and predictable storage, but their length cannot change. Use `List<T>` when the collection grows or shrinks, and use a multidimensional array when rectangular indexing matters.

## How It Works

An array exposes `Length`, stores elements in index order, and performs bounds checks. Reference-type arrays store references; value-type arrays store values inline. Arrays implement `IEnumerable<T>` and common collection interfaces.

## Application

Use arrays for fixed-size buffers, interop boundaries, lookup tables, and performance-sensitive contiguous data.

## Common Mistakes

- Accessing an invalid index.
- Assuming multidimensional and jagged arrays have the same layout.
- Resizing by repeatedly copying without considering `List<T>`.
- Mutating a reference-type element when a copy was expected.

## Common Interview Questions

### Basic

- What is an array?
- Are arrays zero-based?
- What is the difference between `Length` and `Count`?

### Intermediate

- How do rectangular and jagged arrays differ?
- What is the complexity of indexed access?
- How does an array implement enumeration?

### Advanced

- Why are arrays covariant while generic collections are invariant?
- What runtime failure can an invalid covariant array write produce?
- How do bounds checks affect performance?
- When can `Span<T>` provide a better view over array data?
- How do array-of-struct layouts affect locality?

### Follow-up Questions

- Can an array length change?
- What happens when an array is assigned to another variable?
- How do you copy an array safely?

### Code Prediction

What is printed?

```csharp
int[] first = [1, 2];
int[] second = first;
second[0] = 9;
Console.WriteLine(first[0]);
```

What happens at runtime?

```csharp
object[] values = new string[] { "a", "b" };
values[0] = 42; // ArrayTypeMismatchException
```

Arrays are covariant for reference types, but the runtime still checks the actual array element type. Generic collections such as `List<T>` are invariant, so this conversion is not allowed for `List<string>` and `List<object>`.

## Practical Tasks

- Implement a rotation operation in place.
- Compare rectangular and jagged array access patterns.
- Measure copying versus indexed processing.

## Readiness Criteria

Explain array storage, indexing, copying, rectangular versus jagged shapes, and when a resizable or span-based alternative is more appropriate.

## References

### Microsoft Learn

- [Arrays](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/arrays)
- [System.Array](https://learn.microsoft.com/dotnet/api/system.array)
