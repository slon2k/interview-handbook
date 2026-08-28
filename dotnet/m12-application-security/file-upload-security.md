# File-Upload Security

## Definition

File uploads (mechanically covered in Module 8) introduce several security risks beyond the size/streaming concerns covered there: a malicious file disguised as something safe, a filename crafted to escape the intended storage location, or an uploaded file served back in a way that lets it execute as code.

```csharp
// Multiple layered checks — no single one is sufficient alone
if (file.Length > MaxSize) return BadRequest();
if (!AllowedExtensions.Contains(Path.GetExtension(file.FileName))) return BadRequest();
if (!IsActuallyAnImage(file)) return BadRequest(); // inspect content, don't trust the extension or claimed Content-Type
var safeName = $"{Guid.NewGuid()}{Path.GetExtension(file.FileName)}"; // never use the client-supplied filename directly
```

## Alternatives & Trade-offs

Trusting the client-supplied filename, extension, and `Content-Type` header is simplest but trivially bypassed — none of these are verified by the server, and an attacker controls all three. Actually inspecting file content (checking magic bytes/signatures, or running a dedicated malware scan) is more work but is the only way to know what was actually uploaded, rather than what it claims to be.

## How It Works

### Extension and Content-Type are claims, not facts

```csharp
// An attacker can name a file "photo.jpg" and set Content-Type: image/jpeg
// while the actual bytes are an executable or a script — neither check alone proves anything
if (file.ContentType == "image/jpeg") { /* NOT sufficient verification */ }
```

```csharp
// Checking the file's actual magic bytes/signature is a real verification step
byte[] header = new byte[4];
await using var stream = file.OpenReadStream();
await stream.ReadAsync(header);
bool isRealJpeg = header[0] == 0xFF && header[1] == 0xD8; // JPEG's actual binary signature
```

### Never use the client-supplied filename for storage

```csharp
// VULNERABLE: a crafted filename like "../../wwwroot/app.dll" could escape the intended storage directory
var path = Path.Combine(uploadsFolder, file.FileName);

// SAFE: generate a new, random filename server-side; store the original name only as metadata if needed for display
var safeName = $"{Guid.NewGuid()}{Path.GetExtension(file.FileName)}";
var path = Path.Combine(uploadsFolder, safeName);
```

A path-traversal-crafted filename (`../../`) used directly in `Path.Combine` can escape the intended upload directory entirely — generating a fresh, random name server-side eliminates this risk structurally, rather than trying to sanitize every possible traversal pattern.

### Never serve uploaded content from a location that can execute code

```
WRONG: storing uploads inside wwwroot alongside application code/scripts, where a misconfigured
       server might execute an uploaded ".aspx"/".php" file as code instead of serving it as static content

RIGHT: store uploads outside the web root, or in dedicated blob storage, served back via a controlled
       endpoint (or with Content-Disposition: attachment) that never executes the content as code
```

### Scanning for malware, for genuinely sensitive upload surfaces

For high-risk upload scenarios (documents from untrusted external parties), integrating a malware-scanning step (a dedicated antivirus/scanning service) before the file is made available to other users adds a layer no client-side check can provide.

## Application

Validate file size, verify actual content (not just claimed extension/`Content-Type`), generate server-side random filenames rather than trusting client input, and store uploads outside any location where they could be executed as code. Add malware scanning for high-risk upload surfaces.

## Common Mistakes

- Trusting the file extension or `Content-Type` header as proof of what the file actually is.
- Using the client-supplied filename directly for server-side storage, exposing path-traversal risk.
- Storing uploaded files in a location that could be served and executed as application code rather than as static content.
- Validating only file size and extension while skipping actual content inspection for high-risk upload surfaces.

## Common Interview Questions

### Basic
- Why can't a file's extension or `Content-Type` be trusted as proof of its actual content?
- Why shouldn't a client-supplied filename be used directly for server-side storage?

### Intermediate
- What is a magic byte/signature check, and what does it verify that extension checking doesn't?
- Where should uploaded files be stored to avoid the risk of them being executed as code?

### Advanced
- How would you design an upload pipeline for a high-risk scenario (documents from untrusted external parties) including malware scanning?
- How does path traversal via a crafted filename work, and how does generating a random server-side filename eliminate the risk structurally rather than just mitigating it?

### Follow-up Questions
- Is checking a file's magic bytes sufficient on its own to guarantee it's safe?
- Should the original client-supplied filename ever be preserved, and if so, how safely?

### Code Prediction
A client uploads a file named `../../wwwroot/malicious.aspx` with `Content-Type: image/jpeg`. If the server uses `Path.Combine(uploadsFolder, file.FileName)` directly and only checks `Content-Type`, what could go wrong?

## Practical Tasks

- Implement magic-byte verification for an image upload endpoint, rejecting files whose actual content doesn't match the claimed type.
- Refactor an upload endpoint using the client-supplied filename directly into one generating a safe, random server-side filename.
- Design a storage and serving strategy for uploaded files that avoids any risk of them being executed as code.

## Readiness Criteria

Verify uploaded file content rather than trusting claimed metadata, eliminate path-traversal risk via server-generated filenames, and store/serve uploads in a way that can't be executed as code.

## References

### Microsoft Learn

- [Upload files in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/file-uploads)

### Other

- [OWASP: File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
