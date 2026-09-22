# FormData, File Uploads, and Multipart Requests

## Definition

`FormData` represents fields and files in the `multipart/form-data` request format browsers use for file uploads. It lets a client submit a native file input with accompanying metadata without manually serializing file bytes into JSON.

## How It Works

```javascript
const formData = new FormData();
formData.append("avatar", fileInput.files[0]);
formData.append("displayName", displayName);

const response = await fetch("/api/profile/avatar", {
  method: "POST",
  body: formData,
  signal: abortController.signal
});
```

Do not set the `Content-Type` header yourself for a `FormData` body. The browser supplies the multipart boundary; replacing the header with only `multipart/form-data` omits that boundary and produces an invalid request.

`fetch` can cancel an upload with `AbortController`, but it does not expose portable upload-progress events. Use `XMLHttpRequest` only when upload progress is a firm requirement, or design a resumable/direct-to-storage upload flow for large files.

## Application

Validate selected files before upload: client-side checks can improve feedback, but the server must enforce size, type, authorization, and malware-scanning policy. Keep the file in `FormData`, map server validation errors to the related field, and clear or retain the selected file based on whether the request actually succeeded.

## Common Mistakes

- Manually setting `Content-Type` for a `FormData` request.
- Base64-encoding an ordinary file into JSON without a contract reason, increasing payload size and memory pressure.
- Trusting the file extension or browser-reported MIME type as server-side security validation.
- Promising upload progress from `fetch` where the browser does not expose it.

## Common Interview Questions

### Foundation

- What does `FormData` represent?
- Why should a client not manually set the multipart `Content-Type` header?

### Intermediate

- How would you cancel a large upload when the user navigates away?
- Why must a server validate files even after client-side validation?

### Advanced

- When would direct-to-storage or resumable uploads be preferable to sending a file through the application API?

## Practical Tasks

- Submit a file and a text field with `FormData`, then handle a structured server validation response.
- Add cancellation to an upload and ensure an aborted request is not rendered as a failed upload.

## Readiness Criteria

You can submit multipart data correctly, explain `fetch` upload-progress limitations, cancel an in-flight upload, and distinguish client feedback from server-side file validation.

## References

- [MDN: FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [MDN: Using files from web applications](https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications)
