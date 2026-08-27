# Testing HttpClient

## Definition

Code depending on `HttpClient` (or a typed client from `IHttpClientFactory`, see Module 8) is tested by substituting a fake `HttpMessageHandler` — the pipeline component that actually sends the request — so the test controls exactly what response comes back without making a real network call.

```csharp
var handler = new FakeHttpMessageHandler(request =>
    new HttpResponseMessage(HttpStatusCode.OK)
    {
        Content = JsonContent.Create(new { total = 100 })
    });

var client = new HttpClient(handler) { BaseAddress = new Uri("https://api.example.com") };
var paymentClient = new PaymentGatewayClient(client);
```

## Alternatives & Trade-offs

Mocking `HttpMessageHandler` directly (or via a small hand-written fake) lets you test the calling code's behavior — parsing, error handling, retries — without any real network dependency, keeping the test fast and deterministic. Actually calling a real (even if sandboxed) external API in a test is far slower, flakier, and dependent on network availability and the external service's own uptime — appropriate only for a small number of genuine integration tests, not the bulk of the suite.

## How It Works

### A minimal fake `HttpMessageHandler`

```csharp
public class FakeHttpMessageHandler : HttpMessageHandler
{
    private readonly Func<HttpRequestMessage, HttpResponseMessage> _responder;
    public FakeHttpMessageHandler(Func<HttpRequestMessage, HttpResponseMessage> responder) => _responder = responder;

    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken) =>
        Task.FromResult(_responder(request));
}
```

Because `HttpClient` delegates all actual sending to its `HttpMessageHandler`, substituting the handler intercepts every request without needing to mock `HttpClient` itself (which, notably, doesn't implement an interface and can't be mocked directly with most mocking libraries anyway).

### Testing error handling against a simulated failure response

```csharp
[Fact]
public async Task ChargeAsync_WhenGatewayReturns500_ThrowsPaymentException()
{
    var handler = new FakeHttpMessageHandler(_ => new HttpResponseMessage(HttpStatusCode.InternalServerError));
    var client = new HttpClient(handler) { BaseAddress = new Uri("https://api.example.com") };
    var paymentClient = new PaymentGatewayClient(client);

    await Assert.ThrowsAsync<PaymentException>(() => paymentClient.ChargeAsync(100m));
}
```

### Verifying the outgoing request itself, not just the response handling

```csharp
HttpRequestMessage? capturedRequest = null;
var handler = new FakeHttpMessageHandler(request =>
{
    capturedRequest = request; // inspect what the code under test actually sent
    return new HttpResponseMessage(HttpStatusCode.OK);
});

await paymentClient.ChargeAsync(100m);
Assert.Equal(HttpMethod.Post, capturedRequest!.Method);
Assert.Contains("Bearer", capturedRequest.Headers.Authorization?.ToString());
```

### Testcontainers/WireMock for higher-fidelity HTTP integration tests

For scenarios needing more realistic HTTP-level behavior (real serialization over an actual socket, testing a full retry/resilience pipeline end-to-end) a tool like WireMock.Net can stand up a real, local HTTP server the test controls, closer to a true integration test than a handler substitution.

## Application

Substitute `HttpMessageHandler` for fast, deterministic unit tests of code that consumes `HttpClient`, verifying both the outgoing request shape and the code's handling of various response scenarios (success, various error codes, malformed responses). Reserve real external HTTP calls for a small number of genuine integration tests.

## Common Mistakes

- Attempting to mock `HttpClient` directly with a mocking library, when the correct seam is the underlying `HttpMessageHandler`.
- Making real network calls in what should be a fast unit test, coupling test reliability to external service availability.
- Testing only the happy-path response and never simulating error responses (4xx/5xx, timeouts, malformed bodies) that the calling code needs to handle correctly.
- Not verifying the outgoing request's shape (headers, method, body) when that's actually part of what the test should confirm — code that silently sends the wrong data can still "pass" a test that only checks the parsed response.

## Common Interview Questions

### Basic
- What's the correct seam for substituting `HttpClient`'s behavior in a test?
- Why can't `HttpClient` itself typically be mocked directly with most mocking libraries?

### Intermediate
- How would you test that code correctly handles a `500` response from an external API?
- How would you verify the outgoing request (headers, method, body) that code under test actually sends?

### Advanced
- How would you design tests covering a full retry/resilience pipeline (from Module 8's `HttpClientFactory` topic) without making real network calls?
- When would a tool like WireMock.Net be preferable to a hand-written fake `HttpMessageHandler`?

### Follow-up Questions
- Does substituting `HttpMessageHandler` require changes to the production code under test, if it's already using `IHttpClientFactory`?
- Can the same fake handler simulate different responses for different requests within one test?

### Code Prediction
A test only ever configures its fake `HttpMessageHandler` to return `200 OK`, and the calling code has a bug in its `500`-handling logic. Would this test suite ever catch that bug?

## Practical Tasks

- Write a fake `HttpMessageHandler` and use it to test both success and error-response handling for a typed HTTP client.
- Write a test that captures and asserts on the outgoing request's headers and body, not just the response handling.
- Set up a WireMock.Net-based test for a scenario needing more realistic HTTP behavior than a simple fake handler provides.

## Readiness Criteria

Substitute `HttpMessageHandler` correctly to test `HttpClient`-dependent code without real network calls, test both request shape and response handling, and cover error scenarios as well as the happy path.

## References

### Microsoft Learn

- [Test HTTP clients](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)

### Other

- [WireMock.Net documentation](https://github.com/WireMock-Net/WireMock.Net)
