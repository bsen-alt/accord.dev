# `Accord.create()`

In ACCORD v1.3, the `Accord` class constructor is **private**. To ensure the engine is fully initialized and ready to handle requests immediately, you must use the static `create()` factory method.

---

## Signature

```typescript
const accord = await Accord.create(config: AccordV2Config);
```

**Returns:** `Promise<Accord>`

---

## Why Use `create()`?

Authorization engines depend on external state (Databases, Files). Initialization in v1.3 is asynchronous because it involves:

1.  Connecting to the storage adapter.
2.  Loading all policies from the database.
3.  Compiling policy conditions.

If we used a standard `new Accord()`, your application might try to make a check before the policies have finished loading, resulting in a "False Deny" (Default Deny behavior).

`create()` guarantees that the Promise resolves only when the engine is **System Ready**.

---

## Configuration: `AccordV2Config`

The `create` method accepts a configuration object to customize the engine's behavior.

| Property  | Type              | Required | Description                                                            |
| :-------- | :---------------- | :------- | :--------------------------------------------------------------------- |
| `adapter` | `IStorageAdapter` | **Yes**  | The storage driver (e.g., `PostgresStoreAdapter`, `FileStoreAdapter`). |
| `jit`     | `JITConfig`       | No       | Configuration for Just-In-Time identity provisioning.                  |
| `webhook` | `WebhookConfig`   | No       | Configuration for streaming audit events to external services (v1.3).  |
| `hooks`   | `LifecycleHooks`  | No       | Callbacks to inject logic before or after evaluation.                  |
| `logger`  | `AuditLogger`     | No       | Custom logger implementation (defaults to `ConsoleAuditLogger`).       |

---

## Detailed Parameters

### `adapter` (Required)

The storage driver that persists policies and identities.

```typescript
import { Accord, PostgresStoreAdapter } from "@navirondynamics/accord";

const adapter = new PostgresStoreAdapter({
  connectionString: process.env.DATABASE_URL,
});
```

- **Note:** If you want to use the `FileStoreAdapter` (for local development), you must still explicitly provide it here.

### `jit` (Optional)

**Just-In-Time Provisioning.** Automatically creates users in your database on their first access if they don't exist.

```typescript
jit: {
  enabled: true,
  attributeMapping: {
    role: "$.claims.groups[0]" // Map 'groups' claim to internal 'role' attribute
  },
  defaultStatus: 'active'
}
```

### `webhook` (New in v1.3)

**Observability.** Sends real-time events to external systems (Splunk, Datadog) for audit trails.

```typescript
webhook: {
  url: "https://hooks.your-siem.com/webhook",
  events: ['decision', 'policy_change', 'identity_provision']
}
```

### `hooks` (Lifecycle Hooks)

**Extension Points.** Allows you to inject custom logic into the authorization flow.

```typescript
hooks: {
  beforeDecision: [
    async (ctx) => {
      // Example: Inject Risk Score from an external service
      const risk = await FraudAPI.check(ctx.request.ip);
      ctx.request.context.risk = risk;
    },
  ];
}
```

---

## Usage Example

```javascript
import {
  Accord,
  PostgresStoreAdapter,
  WebhookAuditLogger,
} from "@navirondynamics/accord";

async function initAccord() {
  // 1. Configure Storage
  const adapter = new PostgresStoreAdapter({
    connectionString: process.env.DATABASE_URL,
    connectionLimit: 20,
  });

  // 2. Initialize the Engine
  const accord = await Accord.create({
    adapter,

    // 3. Enable Automatic Provisioning
    jit: {
      enabled: true,
      attributeMapping: { role: "$.groups[0]" },
      defaultStatus: "active",
    },

    // 4. Enable Webhook Observability (v1.3)
    webhook: {
      url: process.env.WEBHOOK_URL,
      events: ["decision", "policy_change", "identity_provision"],
    },

    // 5. Custom Audit Logger
    logger: new WebhookAuditLogger({ url: process.env.WEBHOOK_URL }),
  });

  console.log("Accord initialized and ready.");
  return accord;
}
```

---

## Common Mistakes

### ❌ Using `new Accord()`

In v1.3, this will result in a TypeScript error.

```typescript
const accord = new Accord({ adapter }); // ERROR: Constructor is private
```

### ✅ Using `Accord.create()`

The correct, type-safe initialization method.

```typescript
const accord = await Accord.create({ adapter }); // SUCCESS
```

---

## Related APIs

- **[`accord.check()`](./accord-check.md)** - Enforcement API.
- **[`accord.simulate()`](./accord-simulate.md)** - Dry-run API.
- **[`PostgresStoreAdapter`](../adapters/postgres.md)** - Storage implementation.
