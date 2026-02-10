title: Architecture Overview

description: Understand the core components and design philosophy behind Accord’s modular and scalable architecture.

author: Naviron Dynamics Team

date: 2025-04-05

---

# Architecture Overview

At its heart, **Accord** is designed around modularity, observability, and extensibility—enabling it to adapt seamlessly across diverse technology stacks while maintaining strict governance boundaries.

The architecture follows a layered model where each layer serves a distinct purpose and communicates through well-defined interfaces. This ensures flexibility in deployment modes and integration strategies.

---

## Core Layers of Accord

### 1. Policy Layer

This is where all access rules live—defined declaratively in JSON or YAML format. Policies specify:

- Who (Principal)
- Can do what (Action)
- On which resource (Resource Type)
- Under what context (Conditions)

Policies are version-controlled and can be simulated before enforcement.

### 2. Evaluation Engine

The brain of Accord. It interprets incoming requests and evaluates them against loaded policies in real time.

Features include:

- Just-In-Time (JIT) principal creation
- Multi-policy matching with precedence resolution
- Condition evaluation using attribute-based logic
- Latency tracing per evaluation step

Evaluation decisions are returned in a standardized format containing:

- Effect (`allow` or `deny`)
- Matched policy ID
- Trace metadata for debugging

### 3. Observability Stack

Every interaction leaves a footprint. Accord logs:

- Full decision traces
- Request latency breakdowns
- Policy match reasons
- Principal/resource mappings

These logs can be exported via:

- Console output
- File sinks
- Webhook integrations (e.g., SIEM tools)

### 4. Storage Adapter Layer

Pluggable storage backends allow Accord to persist state such as:

- Principal identities
- Policy versions
- Audit logs

Supported adapters include:

- PostgreSQL (default)
- File-based (for development/testing)
- Custom implementations possible via SDK

### 5. Integration Adapters

Designed for seamless adoption, Accord integrates natively with popular frameworks:

- Express.js
- NestJS
- Fastify
- And others via generic middleware wrappers

Guards/middleware intercept requests, pass relevant context to Accord, and enforce decisions transparently.

---

## Deployment Modes

### Library Mode

Embed Accord directly into your Node.js apps.

````js
const { Accord, PostgresStoreAdapter } = require("@navirondynamics/accord");

const adapter = new PostgresStoreAdapter({ connectionString: "..." });
const accord = await Accord.create({ adapter });

await accord.check("alice", "view", { type: "document" });


Best suited for monoliths or small-scale deployments.

### Server Mode

Run Accord as a standalone HTTP service that handles centralized policy decisions.

Start server:

```bash
npx @navirondynamics/accord serve --adapter postgres
````

Make checks:

```bash
curl -X POST http://localhost:8080/api/v1/check \
  -H "Content-Type: application/json" \
  -d '{"userId":"alice","action":"view","resource":{"type":"document"}}'
```

Ideal for microservice architectures requiring shared authorization logic.

---

## Visualizing Permissions

New in v1.3, Accord offers a **Graph API** endpoint that returns a structured representation of relationships between principals, actions, and resources.

Sample usage:

```bash
curl http://localhost:8080/api/v1/policies/graph
```

Output (example):

```json
{
  "nodes": [
    { "id": "user_alice", "type": "principal" },
    { "id": "doc_123", "type": "resource" }
  ],
  "edges": [{ "from": "user_alice", "to": "doc_123", "action": "view" }]
}
```

This enables integration with visualization tools like D3.js or Neo4j for interactive exploration.

---

## Security Design Principles

Accord adheres to several key security tenets:

- **Least Privilege Enforcement**: Every action must explicitly be allowed.
- **Immutable Policy History**: Once published, policies cannot be mutated—only versioned.
- **Zero Trust Context Handling**: All inputs are sanitized and validated.
- **Transparent Logging**: No silent failures; every denied access attempt is recorded.

Additionally, Accord supports:

- Mutual TLS authentication (in server mode)
- JWT token introspection
- Role mapping from external providers (LDAP, OAuth scopes, etc.)

---

## Scalability & Performance

Accord is engineered for high throughput and low-latency scenarios:

- Asynchronous initialization patterns ensure non-blocking startup
- Caching mechanisms reduce repeated evaluations
- Horizontal scaling supported via multiple instances sharing the same DB backend

Benchmarking targets:

- Sub-millisecond evaluation times for simple policies
- <10ms for complex multi-condition evaluations

---

## Extensibility Points

Want to plug in custom behavior?

Accord exposes extension points for:

- Custom condition evaluators
- Principal enrichment pipelines
- Logging formatters
- Storage adapter interfaces

Developers can extend Accord without modifying core logic, preserving upgrade paths.

---

## Summary Diagram

While we don’t embed images directly here, imagine a diagram showing:

[Principal] --> [Request] --> [Evaluation Engine]

↓

[Policy Store (DB)]

↓

[Observability (Logs/Webhooks)]

This reflects the clean separation of concerns that defines Accord’s robustness and clarity.

---

## Next Steps

Curious about how policies are written?

👉 Read: [Writing Your First Policy](/learn/getting-started/first-policy)

Want to understand how Accord handles edge cases?

👉 See: [Deep Trace Analysis](/learn/advanced/deep-trace)

Need to integrate with your existing stack?

👉 Check: [Framework Integrations](/docs/cli/reference)

Let’s architect trust together—with precision, scale, and confidence.
