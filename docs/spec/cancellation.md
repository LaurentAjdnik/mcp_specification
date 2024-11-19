---
title: Cancellation
type: docs
weight: 5
---

MCP supports request cancellation through a dedicated notification message. This allows either party to signal that they are no longer interested in the result of an in-flight request.

## Cancellation Protocol

When a party wants to cancel a request, they send a "cancelled" notification containing the ID of the request they wish to cancel:

For example, initiating a request:

```json
{
  "jsonrpc": "2.0",
  "id": "abc123",
  "method": "resources/read",
  "params": {
    "uri": "example://large-file"
  }
}
```

… then some time later, cancelling that request:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": "abc123",
    "reason": "User interrupted operation"
  }
}
```

## Handling Cancellation

Upon receiving a cancellation notification, the receiver SHOULD:

1. Stop any processing related to the cancelled request
2. Free any resources associated with that request
3. Not send a response for the cancelled request (even if already computed)

However, due to race conditions and message latency:

- The notification may arrive after processing is complete
- A response may already be in transit when cancellation is received
- Multiple cancellation notifications for the same request ID may be received

## Special Cases

### Initialize Request

The client MUST NOT attempt to cancel its `initialize` request. Cancelling initialization leaves the protocol in an undefined state.

### Long-Running Operations

For long-running operations that support progress notifications, progress notifications SHOULD stop being sent after receiving cancellation.

## Best Practices

1. Implement cancellation handlers for all long-running operations
2. Free resources promptly when receiving cancellation
3. Make cancellation handling idempotent
4. Include meaningful reason strings to aid debugging
5. Log cancellations appropriately for troubleshooting