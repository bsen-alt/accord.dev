# About Accord

Welcome to **Accord**, the architectural agreement layer redefining how systems govern access, enforce policies, and evolve securely in modern environments.

Born out of necessity at [Naviron Dynamics](https://navirondynamics.com), Accord emerged from years of grappling with fragmented authorization models across cloud-native platforms. Traditional Role-Based Access Control (RBAC) and Attribute-Based Access Control (ABAC) were insufficient—too static, too scattered, and often inconsistent.

### The Problem

In today’s distributed architectures:

- Authentication lives in one service.
- Roles are managed elsewhere.
- Authorization logic is sprinkled throughout microservices.
- Compliance becomes an afterthought.

This leads to brittle systems prone to misconfigurations, security gaps, and audit nightmares.

### Enter Accord

Accord introduces a **policy-first identity and access platform** built on three foundational pillars:

#### 1. Policy as Code

Define permissions declaratively using JSON/YAML configurations. Every rule is versioned, testable, and human-readable.

#### 2. Centralized Evaluation

A unified engine evaluates every request against centrally defined policies—no more duplicated logic or inconsistency.

#### 3. Observability & Governance

Every decision comes with full traceability: latency metrics, matched policies, impacted resources—all logged and auditable.

With Accord, you shift authorization from scattered application code to a formalized, governable system interface.

---

## Key Concepts

### Identity

Any actor interacting with your system: users, services, devices, or even scheduled jobs.

### Resource

Entities protected by policies: documents, APIs, databases, Kubernetes pods, etc.

### Action

Operations performed on resources: view, edit, delete, approve, etc.

### Policy

Declarative statements that define whether an identity can perform an action on a resource under certain conditions.

Example policy snippet:

```json
{
  "id": "admin-can-edit-any-document",
  "version": "1.0",
  "effect": "allow",
  "principal": { "role": "admin" },
  "action": ["update"],
  "resource": { "type": "document" }
}
```

---

## Evolution Through Versions

| Version | Highlights                                                  |
| ------- | ----------------------------------------------------------- |
| v1.1    | Library-mode core engine                                    |
| v1.2    | Introduced server mode and JIT provisioning                 |
| v1.3    | Policy simulation, visual graphs, rollback, impact analysis |

Today, Accord powers secure interactions in financial services, healthcare platforms, and SaaS ecosystems worldwide.

---

## Who Uses Accord?

Organizations adopting Accord typically fall into these categories:

- Teams needing fine-grained access control beyond RBAC
- Enterprises seeking compliance-ready authorization layers
- DevOps engineers looking to unify auth logic across stacks
- Product teams wanting to decouple permissions from business logic

Whether you're securing internal tools or exposing customer-facing APIs, Accord scales with your needs.

---

## Open Source First

From day one, Accord has been developed in the open under the ISC License. Contributions are welcomed—from bug reports to feature proposals to documentation improvements.

Check us out on [GitHub](https://github.com/navirondynamics/accord) and join our vibrant community on [Discord](https://discord.gg/accord).

---

## Get Started Today

Ready to bring structured governance to your system?

👉 Explore our [Getting Started Guide](/learn/getting-started/installation)

👉 Try the [Interactive Playground](/accord/playground)

👉 Dive into [API Docs](/docs)

Let’s build safer, smarter systems—together.
