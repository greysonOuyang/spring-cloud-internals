# Upstream issue investigations

Each issue directory is an engineering notebook, not a copy of the GitHub discussion.

## Selection score

- Internal learning value: 30%
- Upstream usefulness: 25%
- Reproducibility: 20%
- Realistic merge probability: 15%
- User/production impact: 10%

Prefer work that is small enough to finish but deep enough to expose a runtime mechanism.

## Rules

1. Check for an existing linked/open PR before implementation.
2. Reproduce against current upstream `main`, not only the reporter's old version.
3. Capture the execution path before editing code.
4. Add a regression test that fails before the fix whenever feasible.
5. Avoid speculative API design until a maintainer direction exists.
6. Use DCO signed commits (`git commit -s`) for upstream work.
7. Keep the knowledge repository separate from upstream patches.

## Initial references already occupied by existing PRs

- Gateway #4243 — Local response cache recreated on route refresh; useful for filter/cache lifecycle study, but upstream PR #4244 already exists.
- Gateway #4270 — immutable `ReadOnlyHttpHeaders` failure in `NettyRoutingFilter`; upstream PR #4272 already exists.
- Gateway #4212 — trailing slash matching after Spring Framework 7 change; upstream PR #4245 already exists.
- Gateway #4224 — gRPC trailers-only status details; upstream PR #4229 already exists.
- Gateway #3278 — route-specific CORS with Host predicates; upstream PR #4142 already exists.

These remain reading material, not duplicate contribution targets.

## First active investigation

- Gateway #4168 — standard Spring MVC `ServerResponse` exposes read-only headers after build while Gateway after filters assume mutable response headers. No corresponding PR found during triage; current-main reproduction and lifecycle analysis are active in `issues/gateway-4168/`.

## Candidate deliberately not selected

- Gateway #3843 — streaming/chunked request bodies bypass the existing `RequestSize` header check. It initially scored highly, but maintainer Spencer Gibb explicitly classified the requested behavior as an enhancement rather than a bug fix and asked for documentation changes; PR #3844 is still open. It remains a useful WebFlux/DataBuffer learning case, but duplicating work would ignore current maintainer direction.
