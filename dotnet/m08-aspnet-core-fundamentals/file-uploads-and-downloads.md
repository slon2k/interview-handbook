# File Uploads and Downloads

## Definition

ASP.NET Core represents an uploaded file as `IFormFile` (buffered, simple to use) or supports streaming uploads directly for large files without buffering the whole thing in memory first. Downloads are served via `FileResult`/`Results.File`, streaming file content back to the client with the correct `Content-Type` and `Content-Disposition` headers.

```csharp
app.MapPost("/upload", async (IFormFile file) =>
{
    using var stream = file.OpenReadStream();
    // process the stream
    return Results.Ok(new { file.FileName, file.Length });
});
```

## Alternatives & Trade-offs

`IFormFile` is simple and sufficient for small-to-moderate uploads (the framework buffers the file, in memory or to a temp file depending on size, before your code runs). For large files, buffering everything before processing wastes memory and delays processing; **streaming** the request body directly avoids buffering the whole file, at the cost of more manual handling of multipart form parsing.

## How It Works

### Simple upload with `IFormFile`

```csharp
app.MapPost("/upload", async (IFormFile file) =>
{
    if (file.Length == 0) return Results.BadRequest("Empty file");
    if (file.Length > 10 * 1024 * 1024) return Results.BadRequest("File too large");

    var path = Path.Combine("uploads", Path.GetRandomFileName());
    await using var destination = File.Create(path);
    await file.CopyToAsync(destination);
    return Results.Ok(new { file.FileName, file.Length });
});
```

### Configuring size limits

```csharp
builder.Services.Configure<FormOptions>(options =>
{
    options.MultipartBodyLengthLimit = 50 * 1024 * 1024; // 50 MB
});
```

Without an explicit limit, a very large upload can consume excessive memory/disk or be used as a denial-of-service vector.

### Streaming large uploads without buffering

```csharp
app.MapPost("/upload-large", async (HttpRequest request) =>
{
    if (!request.HasFormContentType) return Results.BadRequest();

    var reader = new MultipartReader(GetBoundary(request.ContentType!), request.Body);
    MultipartSection? section;
    while ((section = await reader.ReadNextSectionAsync()) != null)
    {
        // process section.Body as a stream directly, without ASP.NET Core buffering the whole file first
    }
    return Results.Ok();
});
```

This avoids the buffering `IFormFile` does automatically, which matters for very large files where memory/disk overhead of buffering would be significant.

### Downloads

```csharp
app.MapGet("/reports/{id:int}/download", (int id) =>
{
    var stream = GetReportStream(id); // a Stream, not loaded fully into memory
    return Results.File(stream, "application/pdf", $"report-{id}.pdf");
});
```

`Results.File` (or `FileStreamResult` in controllers) streams the content back to the client and sets `Content-Type` and `Content-Disposition` correctly, rather than requiring the file to be fully loaded into a byte array first.

## Application

Use `IFormFile` for typical small-to-moderate file uploads (profile pictures, documents up to a few tens of megabytes). Use streaming APIs for large file uploads where buffering the whole file would be wasteful. Always enforce explicit size limits and validate content type/extension server-side (never trust the client-supplied `Content-Type` alone) before processing an uploaded file.

## Common Mistakes

- Not setting an explicit upload size limit, leaving the endpoint vulnerable to resource-exhaustion from oversized uploads.
- Trusting the client-supplied file extension or `Content-Type` header to determine what kind of file was actually uploaded, without validating the actual content (a classic file-upload security issue — see Module 12).
- Loading an entire large file into memory (`byte[]`) for both upload processing and download serving, when streaming would avoid the memory spike.
- Not sanitizing or randomizing the stored filename, risking path traversal or overwriting existing files if the original filename is used directly.

## Common Interview Questions

### Basic
- What is `IFormFile`, and how do you read its contents?
- How do you serve a file for download in ASP.NET Core?

### Intermediate
- Why would you use streaming instead of `IFormFile` for very large uploads?
- What configuration controls the maximum allowed upload size?

### Advanced
- How would you validate that an uploaded file's actual content matches its claimed type, rather than trusting the `Content-Type` header?
- How would you design an upload endpoint that streams a very large file directly to blob storage without buffering it fully in the web server's memory?

### Follow-up Questions
- Does `Results.File` load the entire file into memory before sending it?
- What's the security risk of using a client-supplied filename directly for server-side storage?

### Code Prediction
An upload endpoint accepts an `IFormFile` with no configured size limit and no `MultipartBodyLengthLimit` override. A client uploads a 2 GB file. What's the practical risk to the server, and what configuration would mitigate it?

## Practical Tasks

- Implement an upload endpoint with explicit size limits and basic content-type validation beyond trusting the client header.
- Implement a streaming upload for large files using `MultipartReader`, avoiding full in-memory buffering.
- Implement a download endpoint that streams file content rather than loading it fully into a byte array first.

## Readiness Criteria

Implement both buffered and streaming upload handling appropriately, enforce size and content-type validation, and stream downloads without unnecessary full in-memory buffering.

## References

### Microsoft Learn

- [Upload files in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/file-uploads)
- [Return binary data (file results)](https://learn.microsoft.com/aspnet/core/mvc/models/file-uploads#special-storage-and-naming-mechanisms-safe-storage-file-names)
