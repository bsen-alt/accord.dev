import { Play, ShieldCheck, Zap, Activity, Terminal } from 'lucide-react';

# `accord.simulate()`

**`simulate`** allows you to evaluate authorization requests safely against a **Mock Identity** without touching your database or triggering audit logs. It is the "Sandbox Mode" for authorization.

---

## Signature

```typescript
const decision = await accord.simulate(
  identity,  // (Object) The mock identity
  action,    // (String) The action to perform
  resource,  // (Object) The target resource
  context?    // (Object) Additional context
);
```

**Returns:** `Promise<DecisionTrace>`

---

## Parameters

### `identity` (Required)

The mock identity object. Unlike `check()` which takes a `userId` and looks up the user, `simulate()` expects the full identity object defined directly in the code.

**Required Properties:**

- `id` (string): A unique identifier for the mock user.
- `type` (`'user' | 'service' | 'system' | 'agent'`): The entity type.
- `status` (`'active' | 'suspended' | 'revoked'): The account state.
- `attributes` (object): Custom attributes (e.g., `role`, `department`, `clearance`).

```javascript
const mockUser = {
  id: "alice",
  type: "user",
  status: "active",
  attributes: { role: "finance", clearance: "high" },
};
```

### `action` (Required)

The operation the identity is attempting to perform.

- _Example:_ `'create'`, `'delete'`, `'read'`.

### `resource` (Required)

The target object being accessed.

- `type`: The kind of resource (e.g., `'invoice'`).
- `id`: The specific instance ID.
- `attributes`: Data associated with the resource (e.g., `amount`, `status`).

### `context` (Optional)

Additional data that influences the policy condition evaluation.

- _Examples:_ `ip`, `time`, `riskScore`, `geo`.

---

## Behavior: What Happens Under the Hood?

1.  **No Database Lookup:** The engine **bypasses** the `IdentityResolver`.
2.  **No Writes:** It evaluates the logic but **does not** save the result or trigger `afterDecision` audit hooks.
3.  **Current Policies:** It evaluates against the policies **currently loaded in memory**. It does not accept a raw policy object as an argument.

**This means you cannot pass a "Draft Policy" to `simulate`.** You must ensure the target policy is already loaded in the engine before calling simulate.

---

## Usage Example: Testing a Dangerous Policy

You are writing a new policy to allow "Managers" to delete "High Value Transactions" (> $10,000). You want to test a scenario where the amount is exactly $10,000 (the edge case) before saving the policy.

```javascript
const manager = {
  id: "manager_01",
  type: "user",
  status: "active",
  attributes: { role: "manager" },
};

// The edge case: Transaction is EXACTLY 10,000
const highValueTx = {
  type: "transaction",
  id: "tx_999",
  attributes: { amount: 10000 },
};

const result = await accord.simulate(manager, "delete", highValueTx);

if (result.decision === "allow") {
  console.warn(
    "Warning: Policy allows deletion of transactions valued at exactly 10,000. Adjust threshold.",
  );
  // Do not deploy until policy is fixed.
}
```

---

## Usage Example: Debugging a Complex Condition

You have a complex policy that denies access if `riskScore > 90`. You want to simulate different risk levels without changing the user's actual risk score in the database.

```javascript
const user = {
  id: 'alice',
  type: 'user',
  const status: 'active',
  attributes: { role: 'finance' }
};

const resource = { type: 'report' };

// Scenario 1: Low Risk
const lowRiskContext = { riskScore: 20 };
const result1 = await accord.simulate(user, 'view', resource, lowRiskContext);
// -> allow

// Scenario 2: High Risk
const highRiskContext = { riskScore: 95 };
const result2 = await accord.simulate(user, 'view', resource, highRiskContext);
// -> deny
```

---

## Comparison: `simulate()` vs `check()`

| Feature             | `check()`                  | `simulate()`                 |
| :------------------ | :------------------------- | :--------------------------- |
| **Input**           | `userId` (String)          | `identity` (Object)          |
| **Identity Source** | Database / JIT             | Argument passed in function  |
| **Side Effects**    | Writes logs, Updates stats | None (Pure Calculation)      |
| **Primary Use**     | Production Enforcement     | Testing & "What-If" Analysis |
| **Latency**         | ~1-2ms                     | < 1ms (Skips DB Lookup)      |

---

## Best Practices

### 1. Keep Tests Isolated

When writing unit tests for your app, prefer `simulate()` if possible. It keeps your tests fast and prevents accidental database pollution during testing.

### 2. Do Not Use for Production

Never use `simulate()` in your actual request flow. It is not optimized for production traffic.

- _Bad:_ `if (isTest) accord.simulate(...) else accord.check(...)`.
- _Good:_ Always use `check()` in app code, use `simulate()` in tests.

### 3. Ensure Policy Is Loaded

Since `simulate` uses the currently loaded policies, if you want to test a new policy definition, you must reload the engine first.

```javascript
// Update policy in DB...
await adapter.savePolicy(newPolicy);
await accord.reload();

// Now test it
await accord.simulate(mockUser, action, resource);
```

---

## Related APIs

- **[`accord.check()`](./accord-check.md)**: The production enforcement method.
- **[`Accord.create()`](./accord-create.md)**: Initialization logic.
- **[Policy Reference](./api/policy)**: Structure of the Policy objects used in simulation.
