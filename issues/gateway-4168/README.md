# Gateway #4168 — response mutation after standard `ServerResponse`

Status: **active investigation**

Upstream: `spring-cloud/spring-cloud-gateway#4168`

## Why this is the first active candidate

- No PR referencing #4168 was found in open or closed Gateway pull requests at triage time.
- The report contains a minimal reproducer and is still open on current upstream.
- It crosses a meaningful framework boundary: Spring MVC functional `ServerResponse` lifecycle vs Spring Cloud Gateway's mutable `GatewayServerResponse` lifecycle.
- Scope is bounded enough for a first contribution, but the root cause affects multiple response-mutating after filters, so it is not a trivial one-line patch.

## Baseline

- Gateway upstream/main: `cda1e040dcadec7438d2055cd29e701ea808a1c6`
- Personal fork `main`, `origin/main`, and `upstream/main` were byte-identical before investigation.
- JDK: Temurin 21.0.12
- Maven: 3.9.12
- Contribution branch: `gh-4168-mutable-after-filter-response`

## Current source finding

Spring Framework 7's standard MVC `AbstractServerResponse` freezes response headers at construction with `HttpHeaders.readOnlyHttpHeaders(...)`.

Spring Cloud Gateway's `AbstractGatewayServerResponse`, by contrast, intentionally stores its own mutable `HttpHeaders` copy. Gateway's `AfterFilterFunctions` currently calls mutation methods directly on `response.headers()` in several functions, including add/set/remove/dedupe/rewrite operations.

Therefore the failing invariant is broader than `addResponseHeader`:

> Gateway after-filter response mutation assumes writable response metadata, while `GatewayRouterFunctions` accepts arbitrary Spring `ServerResponse` implementations whose metadata is immutable after build.

The investigation is testing a unified response adaptation boundary rather than patching each header function independently.

## Reproduction

A focused unit regression test was added locally that applies `addResponseHeader` to `ServerResponse.ok().build()`. It is expected to fail on the unmodified baseline with `UnsupportedOperationException`; exact test evidence will be recorded after Maven completes.

## Maintainer interaction strategy

Once the behavior and viable compatibility boundary are verified, upstream communication should include:

1. current-main reproduction;
2. the two response lifecycle models;
3. why changing only `addResponseHeader` is incomplete;
4. a narrow design proposal with regression tests.

Do not open a speculative PR until the response/body preservation semantics are proven locally.
