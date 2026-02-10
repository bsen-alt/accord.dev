# CLI Reference

The Accord CLI (`npx @navirondynamics/accord`) is the primary interface for operating the governance platform. It handles everything from starting the server to managing policy versions and validating syntax.

---

## Global Usage

All commands follow the standard structure:

```bash
npx @navirondynamics/accord <command> [options]
```

````

### Global Options

| Option          | Argument | Description                   |
| :-------------- | :------- | :---------------------------- |
| `-h, --help`    | -        | Display help for the command. |
| `-V, --version` | -        | Output the version number.    |

---

## Operations Commands

These commands are used to start and manage the Platform (Server Mode).

### `serve`

Starts the standalone HTTP Server. This is the primary entry point for running Accord in production or staging environments.

**Syntax:**

```bash
npx @navirordynamics/accord serve [options]
```

**Options:**

| Option           | Argument             | Default                    | Description                            |
| :--------------- | :------------------- | :------------------------- | :------------------------------------- |
| `-a, --adapter`  | `file` \| `postgres` | `file`                     | The storage engine to use.             |
| `-p, --port`     | `number`             | `8080`                     | The port the server listens on.        |
| `--postgres-url` | `url`                | `process.env.DATABASE_URL` | Direct connection string for Postgres. |

**Examples:**

**Start in Production (Postgres):**

```bash
npx @navirondynamics/accord serve --adapter postgres --postgres-url postgres://user:pass@host:5432/accord
```

**Start for Development (File):**

```bash
npx @navirondynamics/accord serve --adapter file
```

**Custom Port:**

```bash
npx @navirondynamics/accord serve --port 3000
```

---

## Development & Validation Commands

These commands are designed for the developer workflow: validating configuration and testing authorization logic locally.

### `validate`

Validates the syntax and structure of a policy file (JSON or YAML) without starting the engine.

**Syntax:**

```bash
npx @navirondynamics/accord validate <path-to-file>
```

**Usage:**

```bash
npx @navirondynamics/accord validate ./config/policies.yaml
```

**Exit Codes:**

- `0`: Success. File is valid.
- `1`: Error. Syntax or Schema violation.

---

### `eval`

Performs a local authorization check (Dry-Run) using the `FileStoreAdapter`. This is useful for CI/CD pipelines or local debugging without spinning up a database.

**Syntax:**

```bash
npx @navirondynamics/accord eval [options]
```

**Options:**

| Option           | Argument | Default                    | Description                          |
| :--------------- | :------- | :------------------------- | :----------------------------------- |
| `-i, --id`       | `string` | -                          | The User ID to check.                |
| `-a, --action`   | `string` | -                          | The action (e.g., `delete`).         |
| `-r, --resource` | `string` | -                          | The resource type.                   |
| `--rid`          | `string` | -                          | The specific Resource ID (optional). |
| `-p, --policy`   | `path`   | `./config/policies.json`   | Path to the policy file.             |
| `-u, --user`     | `path`   | `./config/identities.json` | Path to the identities file.         |

**Usage:**

```bash
npx @navirondynamics/accord eval \
  -i alice \
  -a delete \
  -r invoice \
  --rid inv_123
```

---

## Governance Commands (v1.3)

These commands manage the lifecycle of policies and identities in the Postgres database.

### `policy history`

View the version history of a specific policy. This is essential for auditing "who did what and when."

**Syntax:**

```bash
npx @navirondynamics/accord policy history <policy-id>
```

**Prerequisites:**

- `DATABASE_URL` must be set in the environment variables.
- The `PostgresStoreAdapter` must be in use.

**Output:**

```text
> History for policy: billing_access
> - v1.0 (2023-10-01): allow
> - v1.1 (2023-11-15): added geo-restriction
> - v1.2 (2024-01-20): revoked for terminated users
```

---

### `policy rollback`

Instantly revert a policy to a previous version. This is the "Safety Net" for the v1.3 versioning system.

**Syntax:**

```bash
npx @navirondynamics/accord policy rollback <policy-id> --to <version>
```

**Options:**

| Option          | Argument | Description                    |
| :-------------- | :------- | :----------------------------- |
| `-v, --version` | `string` | The target version to restore. |

**Prerequisites:**

- `DATABASE_URL` must be set.
- Target version must exist in history.

**Usage:**

```bash
# Rollback 'billing_access' to version 1.0
npx @navirondynamics/accord policy rollback billing_access --to 1.0
```

**Result:**

- A new version (e.g., `1.0.1`) is created that replicates the state of `1.0`.
- The active pointer is updated to `1.0.1`.
- The server hot-reloads the policy.

---

## Environment Variables

The CLI relies on environment variables (`.env`) for sensitive data.

| Variable         | Required For         | Description                                  |
| :--------------- | :------------------- | :------------------------------------------- |
| `DATABASE_URL`   | `--adapter postgres` | Connection string for the Postgres database. |
| `WEBHOOK_URL`    | Webhook features     | URL to send audit events to.                 |
| `WEBHOOK_SECRET` | Webhook features     | Secret key for HMAC signing of payloads.     |
| `JIT_ENABLED`    | All modes            | Enables/disables Just-In-Time provisioning.  |

---

## Command Aliases

For convenience, some commands have shorter aliases.

| Full Command      | Alias | Note                     |
| :---------------- | :---- | :----------------------- |
| `policy rollback` | `p r` | (Planned future feature) |

---

## Detailed Guides

For deep dives into specific command usage and configuration, refer to the dedicated documentation:

- **[Server Mode Configuration](./serve.md)** - Detailed setup for production deployment.
- **[Policy Rollback Guide](../learn/advanced/rollback.md)** - Best practices for using rollback in CI/CD.

````
