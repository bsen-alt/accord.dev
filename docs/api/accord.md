import { Server, Terminal, ShieldCheck, Zap, RefreshCw, Database, HardDrive } from 'lucide-react';

# Accord Class Reference

The `Accord` class is the core engine. It is responsible for resolving identities, evaluating policies, and returning authorization decisions.

---

## Initialization

### `Accord.create(config)`

In ACCORD v1.3, the constructor is private. You must use the static factory method `create` to initialize the engine. This ensures all asynchronous setup (like loading policies from the database) completes before the instance is returned.

**Signature:**

```typescript
import { Accord, PostgresStoreAdapter } from "@navirondynamics/accord";

const accord = await Accord.create({
  adapter: new PostgresStoreAdapter({
    connectionString: process.env.DATABASE_URL,
  }),
  jit: { enabled: true, defaultStatus: "active" },
  webhook: { url: "https://hooks.your-siem.com/webhook", events: ["decision"] },
});
```

**Parameters:**

- **`adapter`** (Required): An instance of `IStorageAdapter` (e.g., `PostgresStoreAdapter` or `FileStoreAdapter`).
- **`jit`** (Optional): Configuration for Just-In-Time provisioning.
  - `enabled`: Boolean. Auto-create identities on first check.
  - `attributeMapping`: Mapping rules for external claims (e.g., SAML groups) to internal attributes.
  - `defaultStatus`: Status assigned to provisioned identities (`active` or `suspended`).
- **`webhook`** (Optional): Configuration for sending audit events to external systems.
  - `url`: Endpoint to receive events.
  - `events`: Array of event types to send (`decision`, `policy_change`, `identity_provision`).

**Returns:** `Promise<Accord>`

---

## Methods

### `check(userId, action, resource, context?)`

The primary method for enforcing authorization in production. It resolves the user, evaluates policies, and returns a decision.

**Signature:**

```typescript
const decision = await accord.check(
  "alice", // userId (string)
  "delete", // action (string)
  { type: "invoice" }, // resource (object)
  { ip: "192.168.1.1" }, // context (object, optional)
);
```

**Parameters:**

- `userId` (string): The external ID of the identity (e.g., `sub` from a JWT).
- `action` (string): The action being attempted (e.g., `'create'`, `'delete'`, `'view'`).
- `resource` (object): The target resource.
  - `type` (string): The type of the resource (e.g., `'invoice'`).
  - `id` (string): The unique ID of the specific resource instance.
  - `attributes` (object): Arbitrary key/value pairs associated with the resource.
- `context` (object, optional): Additional request context (e.g., IP address, Time, Risk Score).

**Returns:** `Promise<DecisionTrace>`

- **`decision`**: `'allow'` | `'deny'`.
- **`policy_id`**: The ID of the specific policy that dictated the decision.
- **`reason`**: Human-readable explanation.
- **`trace`**: Detailed execution log (latency, matched policies, resolved attributes).

---

### `simulate(identity, action, resource, context?)`

A "Dry-Run" version of `check`. It evaluates policies against a provided mock identity without hitting the database or writing logs. Essential for testing and debugging.

**Signature:**

```typescript
const mockUser = {
  id: "alice",
  type: "user",
  status: "active",
  attributes: { role: "admin" },
};

const decision = await accord.simulate(
  mockUser,
  "delete",
  { type: "invoice", id: "inv_999" },
  { ip: "10.0.0.1" },
);
```

**Parameters:**

- `identity` (object): The full Identity object. This bypasses the database lookup.

**Returns:** `Promise<DecisionTrace>`

---

### `reload()`

Instructs the engine to re-read policies and identities from the storage adapter. Useful when policies are updated externally (e.g., via the Management API) or when the `FileStoreAdapter` detects file changes.

**Signature:**

```typescript
await accord.reload();
```

**Returns:** `Promise<void>`

---

### `getAdapter()`

Returns the initialized storage adapter instance. Useful for accessing low-level database methods (e.g., `adapter.listPolicies()`).

**Signature:**

```typescript
const adapter = accord.getAdapter();
const policies = await adapter.listPolicies();
```

---

## Interfaces & Types

### `DecisionTrace`

The object returned by `check` and `simulate`.

```typescript
interface DecisionTrace {
  decision: "allow" | "deny";
  policy_id?: string;
  reason?: string;
  trace: {
    matchedPolicies: string[];
    evaluatedPolicies: string[];
    latencyMs: number;
    resolvedAttributes: Record<string, any>;
  };
}
```

### `AccordConfig`

The configuration object passed to `Accord.create`.

```typescript
interface AccordV2Config {
  adapter: IStorageAdapter;
  jit?: JITConfig;
  hooks?: LifecycleHooks;
  webhook?: WebhookConfig;
  logger?: AuditLogger;
}
```

---

## Usage Example: Full Flow

```javascript
import {
  Accord,
  PostgresStoreAdapter,
  WebhookAuditLogger,
} from "@navirondynamics/accord";

async function init() {
  const adapter = new PostgresStoreAdapter({
    connectionString: process.env.DB_URL,
  });

  const accord = await Accord.create({
    adapter,
    jit: { enabled: true },
    webhook: {
      url: process.env.WEBHOOK_URL,
      events: ["decision", "policy_change"],
    },
  });

  // Enforcement
  const allowed = await accord.check("alice", "delete", {
    type: "invoice",
    id: "123",
  });

  if (allowed.decision === "allow") {
    console.log("Access Granted");
  }
}
```

---

## Related APIs

- **[CLI Reference](/docs/cli/serve)**
- **[Adapter Interface](/docs/adapters/interface)**
- **[Policy Reference](/docs/api/policy)**
