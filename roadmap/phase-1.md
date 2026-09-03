# Phase 1 — Gateway-first source map

## Objective

Build enough source-level understanding to make a first meaningful Spring Cloud Gateway contribution without choosing a trivial documentation-only task.

## Runtime paths

### 1. Request → Route
Trace:
`HttpWebHandlerAdapter → RoutePredicateHandlerMapping → RouteLocator → Route`

Questions:
- Where do `RouteDefinition` objects come from?
- When are predicates materialized?
- What is cached, and what is rebuilt on refresh?

### 2. Route → Filter chain
Trace:
`FilteringWebHandler → GlobalFilter/GatewayFilter ordering → DefaultGatewayFilterChain`

Questions:
- When is a filter factory `apply()` executed?
- Which state belongs to a route, and which belongs to a request?
- What changes when the route cache refreshes?

### 3. `lb://service` → concrete instance
Trace:
`ReactiveLoadBalancerClientFilter → LoadBalancerClientFactory → ReactorServiceInstanceLoadBalancer → ServiceInstanceListSupplier`

Questions:
- How does request context reach load-balancing decisions?
- How is supplier composition implemented?
- Where are retry and sticky-session semantics layered?

### 4. Discovery → dynamic routes
Trace:
`DiscoveryClient → DiscoveryClientRouteDefinitionLocator → RouteDefinitionRouteLocator → CachingRouteLocator`

Questions:
- Which events cause rebuilds?
- How are SpEL expressions evaluated?
- What are the security boundaries around expression evaluation?

### 5. Configuration refresh
Trace across Commons/Config:
`remote config/environment change → EnvironmentChangeEvent → rebinding/RefreshScope → bean replacement`

Questions:
- Which properties are rebound in place?
- Which beans are destroyed/recreated?
- What breaks when a type cannot be safely reconstructed?

## Deliverables

Each path gets:
- one source trace;
- one sequence diagram;
- one runnable or focused test-based lab;
- links to real issues whose root cause touches the path.
