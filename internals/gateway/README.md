# Spring Cloud Gateway internals

Primary study surface.

Planned traces:
- request-to-route matching;
- route definition materialization and refresh;
- filter ordering and execution;
- forwarded-header processing;
- local response cache lifecycle;
- WebFlux routing through Reactor Netty;
- Server MVC routing through Spring MVC/HTTP clients;
- `lb://` handoff into Spring Cloud LoadBalancer.
