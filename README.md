# Spring Cloud Internals

A scenario-driven learning and upstream contribution workspace for Spring Cloud.

The goal is not to collect API usage notes. Every topic should end in one or more of:

1. a traced runtime path through real Spring Cloud source;
2. a reproducible lab;
3. an upstream issue investigation;
4. a regression test or upstream pull request;
5. a concise public explanation of the mechanism learned.

## Upstream focus

- `spring-cloud/spring-cloud-gateway` — primary contribution surface.
- `spring-cloud/spring-cloud-commons` — core abstractions: context, discovery, load balancing.
- `spring-cloud/spring-cloud-config` — environment/configuration/refresh lifecycle.

Local forks live beside this repository under `~/DevProjects/` with `origin` pointing to `greysonOuyang/*` and `upstream` pointing to `spring-cloud/*`.

## Learning method

We read source by runtime scenario, not file order. Each trace starts from a real request/event and follows the execution path until the underlying abstraction is understood.

Initial paths:

- Gateway request routing
- Gateway filter chain assembly/execution
- `lb://` resolution and reactive load balancing
- Discovery route generation and refresh
- `EnvironmentChangeEvent` / `RefreshScope` / configuration rebinding
- Config Client bootstrap and remote property loading

See [roadmap/phase-1.md](roadmap/phase-1.md).

## Contribution loop

`triage → reproduce → source trace → root cause → regression test → design → patch → upstream review → merge → internals note`

A PR count is not the objective. The objective is repeated, reviewable contribution in a coherent subsystem.

See [contributing/upstream-workflow.md](contributing/upstream-workflow.md) and [issues/README.md](issues/README.md).

## Current status

- Public knowledge repository created.
- Gateway, Commons and Config forks created and cloned.
- JDK 21 and Maven available locally.
- First issue triage is in progress. Issues with an active upstream PR are kept as learning references, not duplicated.
