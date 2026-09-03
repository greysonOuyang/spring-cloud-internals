# Server MVC response lifecycle: Spring MVC vs Gateway

This note comes from investigating Spring Cloud Gateway issue #4168. It is intentionally mechanism-first: the interesting part is not the exception, but why two `ServerResponse` implementations expose different mutation semantics.

## The boundary

Spring Cloud Gateway Server MVC builds on Spring MVC functional endpoints. Its filters therefore receive the Spring interface:

```java
ServerResponse
```

That interface exposes `statusCode()`, `headers()`, `cookies()`, and `writeTo(...)`, but it does **not** promise that the returned metadata is mutable.

### Standard Spring MVC response

Spring Framework's `AbstractServerResponse` freezes headers at construction:

```java
this.headers = HttpHeaders.readOnlyHttpHeaders(headers);
```

By the time an `after` response processor receives `ServerResponse.ok().build()`, response metadata is already immutable. `HandlerFilterFunction.ofResponseProcessor(...)` simply calls the processor with the response returned by the next handler; it does not create a mutable editing phase.

The lifecycle is roughly:

```text
ServerResponse builder
  → response constructed
  → headers become read-only
  → handler returns response
  → after response processor receives response
  → response.writeTo(...)
```

So `response.headers().add(...)` in an after processor is outside Spring MVC's normal builder mutation window.

### Gateway response

Gateway introduced `GatewayServerResponse` for Server MVC. Its `AbstractGatewayServerResponse` deliberately uses a mutable copy:

```java
this.headers = HttpHeaders.copyOf(headers);
```

It also exposes mutable status via `setStatusCode(...)`.

Its lifecycle is therefore:

```text
GatewayServerResponse builder
  → response constructed with mutable metadata copy
  → Gateway after filters mutate metadata
  → response.writeTo(...)
```

This is why the same response filter works for proxied Gateway responses but fails for a standard response returned by a custom handler.

## Historical origin of the mismatch

The original Server MVC implementation guarded response-header mutation with:

```java
if (response instanceof GatewayServerResponse res) {
    res.headers()....
}
```

Commit `1863ade599ce14f01845863aac00dafeef7cae2f` on 2023-07-10 removed that type check for `addResponseHeader` and `setResponseHeader`, describing the cast as unnecessary. The change was type-correct because `ServerResponse.headers()` returns `HttpHeaders`, but it was not lifecycle-correct because `HttpHeaders` may be read-only.

This is a useful source-reading lesson: Java type compatibility does not imply mutation-contract compatibility.

## Why `ServerResponse.from(response)` is not a generic repair

Spring Framework's `DefaultServerResponseBuilder(ServerResponse other)` copies:

- status;
- headers;
- cookies.

It does not copy the original response's entity/rendering/streaming/custom `writeTo` behavior. Calling `ServerResponse.from(original).build()` therefore creates a new metadata-equivalent response with an empty write function, not a semantic clone of an arbitrary response body.

Gateway's corresponding builder has the same fundamental limitation for generic adaptation.

Therefore a correct arbitrary-response adapter would need to preserve the original `writeTo` semantics, not merely copy response metadata.

## Design fork exposed by #4168

There are two coherent contracts:

### A. Gateway-only mutable response filters

Response-mutating Gateway filters operate only on `GatewayServerResponse`. Standard Spring responses remain immutable and are either ignored or rejected with a clear contract.

Advantages:
- matches the original Gateway Server MVC implementation;
- small and predictable;
- consistent with `setStatus`, which already checks `GatewayServerResponse`;
- avoids inventing a response-body delegation layer.

Cost:
- custom handlers must return `GatewayServerResponse` if they want Gateway response mutation.

### B. Arbitrary `ServerResponse` adaptation

Gateway accepts any Spring MVC functional response and wraps it in a mutable response decorator before after-filter processing.

Advantages:
- intuitive for users composing normal Spring MVC functional handlers with Gateway filters.

Costs:
- the decorator must preserve entity/rendering/streaming/custom write behavior;
- it must arbitrate status/header/cookie writes between the original response and mutated metadata;
- error-handler and async behavior need explicit coverage;
- this is a broader API contract than a one-line header fix.

## Current contribution decision

Do not silently choose B inside a small bug fix. The upstream investigation comment asks maintainers to confirm whether arbitrary custom-handler `ServerResponse` values are intended to be first-class inputs to Gateway response-mutating filters. Local regression work can continue while that semantic direction is resolved.
