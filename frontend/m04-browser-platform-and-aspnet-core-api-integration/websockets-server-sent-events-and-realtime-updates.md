# WebSockets, Server-Sent Events, and Realtime Updates

## Definition

Realtime updates deliver information after the initial HTTP response. Polling repeatedly asks for changes, Server-Sent Events (SSE) stream messages from server to browser, and WebSockets provide a persistent two-way connection.

## Alternatives & Trade-offs

- **Polling** is simple and works with ordinary HTTP infrastructure, but can waste requests and delay updates.
- **SSE** is server-to-client only, reconnects automatically in the browser, and suits notifications, progress, and feeds.
- **WebSockets** support low-latency two-way messaging, but require connection lifecycle, reconnection, and authorization design.

## How It Works

```javascript
const stream = new EventSource("/api/notifications");
stream.addEventListener("message", event => {
  const notification = JSON.parse(event.data);
  renderNotification(notification);
});

stream.addEventListener("error", () => {
  // The browser normally attempts to reconnect; close explicitly when the feature unmounts.
});
```

An SSE connection is one-way: client commands remain ordinary HTTP requests. A WebSocket is bidirectional, so the application must define message types, authentication on connection, reconnection behavior, ordering expectations, and what happens when a client misses messages. ASP.NET Core applications commonly expose realtime hubs through SignalR, which abstracts some transport and reconnect concerns without removing those design decisions.

## Application

Start with polling when freshness requirements are modest. Choose SSE for server-pushed updates where the browser does not need to send messages on the same channel. Choose WebSockets or SignalR for collaborative editing, chat, or interactive presence. Treat a realtime message as a hint to reconcile state, not automatically as permanent truth if the feature has ordering or authorization constraints.

## Common Mistakes

- Choosing WebSockets for a requirement that a short polling interval would satisfy more reliably.
- Leaving a connection open after a page or component no longer needs it.
- Assuming messages are guaranteed to arrive exactly once or that reconnecting fills every gap automatically.
- Treating realtime channels as exempt from authentication and authorization rules.

## Common Interview Questions

### Foundation

- What is the difference between polling, SSE, and WebSockets?
- When is SSE preferable to a WebSocket?

### Intermediate

- What lifecycle work is needed when a realtime connection disconnects or a user navigates away?
- Why might a UI refetch after receiving a realtime notification?

### Advanced

- How would you design a collaborative feature that handles reconnects, missed messages, and authorization changes?

## Practical Tasks

- Choose polling, SSE, or WebSockets for a dashboard, notification feed, and chat feature; justify each choice.
- Design a reconnect and cleanup policy for a SignalR or WebSocket-driven feature.

## Readiness Criteria

You can choose a realtime mechanism based on communication direction and freshness needs, explain its lifecycle trade-offs, and account for reconnection and authorization behavior.

## References

- [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [MDN: WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Microsoft Learn: ASP.NET Core SignalR](https://learn.microsoft.com/aspnet/core/signalr/introduction)
