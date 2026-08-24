# Adapter Pattern

## Definition

The Adapter pattern converts the interface of an existing class into another interface that callers expect, letting incompatible types work together without modifying either one.

```csharp
public interface IPaymentProcessor
{
    Task<bool> ChargeAsync(decimal amount);
}

// Third-party SDK we don't control, with an incompatible shape
public sealed class LegacyPaymentSdk
{
    public int Pay(double amountInCents) => 0; // returns a status code, not a bool
}

public sealed class LegacyPaymentAdapter : IPaymentProcessor
{
    private readonly LegacyPaymentSdk _sdk;
    public LegacyPaymentAdapter(LegacyPaymentSdk sdk) => _sdk = sdk;

    public Task<bool> ChargeAsync(decimal amount)
    {
        var statusCode = _sdk.Pay((double)(amount * 100));
        return Task.FromResult(statusCode == 0);
    }
}
```

## Alternatives & Trade-offs

An adapter isolates incompatibility at one seam instead of scattering conversion logic across every call site. Compared to modifying the third-party class directly (often impossible) or changing your own interface to match the vendor's shape (couples your domain to a vendor), the adapter keeps your abstraction stable while absorbing the vendor's quirks. The cost is one extra class per integration and the risk of the adapter itself growing translation logic that deserves its own tests.

## How It Works

```csharp
public sealed class OrderService
{
    private readonly IPaymentProcessor _paymentProcessor; // depends only on the app's own interface
    public OrderService(IPaymentProcessor paymentProcessor) => _paymentProcessor = paymentProcessor;

    public Task<bool> CheckoutAsync(decimal amount) => _paymentProcessor.ChargeAsync(amount);
}
```

`OrderService` never sees `LegacyPaymentSdk`, its cents-based API, or its integer status codes — only `IPaymentProcessor`. If the vendor SDK is later replaced, only `LegacyPaymentAdapter` (or its replacement) changes.

## Application

Use an adapter when integrating a third-party library, legacy code, or external API whose interface doesn't match your application's abstractions — payment gateways, cloud storage SDKs, legacy in-house libraries being gradually replaced.

## Common Mistakes

- Letting vendor-specific types (exceptions, status codes, enums) leak past the adapter into application code.
- Skipping the adapter and calling the third-party SDK directly from multiple services, duplicating conversion logic and making a future SDK swap expensive.
- Writing an adapter with no unit tests, even though its conversion logic (unit conversion, status code mapping) is exactly the kind of logic prone to off-by-one and mapping errors.

## Common Interview Questions

### Basic
- What problem does the Adapter pattern solve?
- How does Adapter differ from Decorator?

### Intermediate
- Why is it risky to let vendor-specific types leak past an adapter?
- When would you introduce an adapter versus just changing your application's interface to match the vendor?

### Advanced
- How would you test an adapter that wraps a third-party SDK without calling the real external service?
- How does Adapter support replacing a vendor dependency later with minimal blast radius?

### Follow-up Questions
- Can an adapter wrap more than one incompatible class behind a single interface?
- Is Adapter still useful if you control both interfaces being connected?

### Code Prediction
Given `LegacyPaymentSdk.Pay` returning `0` for success, what would `LegacyPaymentAdapter.ChargeAsync` return if the SDK returned a non-zero status code, and why is it important that this mapping lives inside the adapter rather than in `OrderService`?

## Practical Tasks

- Write an adapter for a fictional third-party logging SDK so it conforms to your application's `ILogger`-like interface.
- Unit test an adapter's conversion logic using a fake or stub for the wrapped SDK.
- Identify vendor-specific leakage in a given integration and refactor it behind a proper adapter.

## Readiness Criteria

Explain the problem Adapter solves and how it differs from Decorator, design an adapter that fully isolates a vendor's shape, and test the adapter's conversion logic in isolation.

## References

### Microsoft Learn

- [Adapter pattern](https://learn.microsoft.com/dotnet/standard/design-patterns/adapter)
