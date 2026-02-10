import { Server, Database, Terminal, PlugZap, Globe } from 'lucide-react';

# Server Mode Configuration

**Server Mode** turns Accord from a library into a standalone HTTP service. It acts as a centralized "Governance Plane" that your applications—regardless of language—can query for authorization decisions.

---

## Quick Start

The fastest way to start the platform is using the CLI included in the package.

```bash
# 1. Install the package
npm install @navirondynamics/accord

# 2. Start the server using File Storage (Default)
npx @navirondynamics/accord serve
```

The server will start on the default port `8080` and load policies from the current working directory (`./config/policies.json`).

---

## Configuration

The server is highly configurable via **Environment Variables** and **CLI Flags**.

### Environment Variables (.env)

Create a `.env` file in your project root (or set them in your CI/CD pipeline).

| Variable         | Required      | Description                                                   |
| :--------------- | :------------ | :------------------------------------------------------------ |
| `DATABASE_URL`   | Postgres Only | Connection string for the Postgres adapter.                   |
| `PORT`           | No            | Port for the HTTP server. Default: `8080`.                    |
| `JIT_ENABLED`    | No            | Enable "Just-In-Time" identity provisioning. Default: `true`. |
| `WEBHOOK_URL`    | No            | URL to send audit events (e.g., to Splunk or Datadog).        |
| `WEBHOOK_SECRET` | No            | HMAC secret for signing webhook payloads.                     |

### CLI Flags

```bash
npx @navirondynamics/accord serve [options]
```

| Flag             | Argument             | Description                                                |
| :--------------- | :------------------- | :--------------------------------------------------------- |
| `-a, --adapter`  | `file` \| `postgres` | The storage engine to use.                                 |
| `-p, --port`     | `number`             | Override the default port.                                 |
| `--postgres-url` | `url`                | Directly pass the Postgres connection string (skips .env). |

**Example:**

```bash
npx @navirondynamics/accord serve \
  --adapter postgres \
  --postgres-url "postgres://user:pass@host:5432/db" \
  --port 3000
```

---

## Storage Adapters

The server must be initialized with a storage adapter to persist policies and identities.

### 1. File Adapter (Default/Local)

Best for: Local development, testing, or simple applications.

**Configuration:**

- **CLI:** `--adapter file` (Default).
- **Data:** Reads/writes `./config/policies.json` and `./config/identities.json`.

**Usage:**

```bash
npx @navirondynamics/accord serve --adapter file
```

### 2. Postgres Adapter (Production)

Best for: Enterprise scale, multi-instance deployment, and Versioning (v1.3+).

**Configuration:**

- **CLI:** `--adapter postgres`.
- **Prerequisite:** A running Postgres instance with the v1.3 schema applied.

**Usage:**

```bash
# Set DATABASE_URL in .env
export DATABASE_URL="postgres://user:password@localhost:5432/accord"

npx @navirondynamics/accord serve --adapter postgres
```

> **Note:** When using Postgres in Server Mode, the server automatically manages database connections via a pool.

---

## APIs Provided

When the server is running, it exposes two main categories of HTTP endpoints:

### 1. The Decision API

Used by your applications to ask "Can I do X?".

**Endpoint:** `POST /api/v1/check`

**Request Body:**

```json
{
  "userId": "alice",
  "action": "delete",
  "resource": { "type": "invoice", "id": "123" },
  "context": { "ip": "192.168.1.1" }
}
```

**Response:**

```json
{
  "decision": "allow",
  "policy_id": "policy-invoice-delete",
  "reason": "Explicit allow policy matched",
  "trace": { ... }
}
```

### 2. The Management API

Used by admins, CI/CD pipelines, or internal tools to manage policies and identities.

- `GET /api/v1/policies` - List all policies.
- `POST /api/v1/policies` - Create a new policy.
- `DELETE /api/v1/policies/:id` - Delete a policy.
- `POST /api/v1/policies/:id/rollback` - Rollback a policy to a previous version.

---

## Hot Reloading

One of the primary benefits of Server Mode is the ability to update governance without restarting the server process.

**How it works:**

1.  **Admin Action:** An admin pushes a new policy via `POST /api/v1/policies`.
2.  **Persistence:** The server saves the new version to the database.
3.  **Signal:** The server triggers a hot-reload event.
4.  **Update:** All in-memory policies are refreshed from the database instantly.
5.  **Result:** The decision API immediately uses the new logic for all subsequent requests.

**Zero-Downtime:** Rolling back a broken policy or pushing an emergency fix is a simple API call, not a deployment process.

---

## Next Steps

- **[API Reference](/docs/api/decision-check)**
- [Postgres Adapter Setup](/docs/adapters/postgres)
