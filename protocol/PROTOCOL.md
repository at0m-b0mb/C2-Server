# LabC2 Wire Protocol — v1

This is the contract that every server implementation and every agent
implementation must satisfy. If your server speaks this protocol and your
agent speaks this protocol, they interoperate — regardless of language.

## 1. Transport

- **Protocol:** HTTP/1.1 or HTTP/2 over TLS 1.2+.
- **Content-Type:** `application/json; charset=utf-8` on all requests and responses.
- **Base path:** Server-defined, but `/api/v1` is the conventional default.

## 2. Authentication

Two principal types:

### 2.1 Operator (human)

Operators authenticate to the server via:

1. `POST /api/v1/auth/login` with `{"username":"...", "password":"...", "totp":"123456"}`.
2. Server returns `{"session_token":"...","expires_at":"..."}`.
3. Subsequent operator requests send `Authorization: Bearer <session_token>`.

Sessions expire after 1 hour of inactivity.

### 2.2 Agent (program)

Agents authenticate with a long-lived pre-shared token issued by an operator:

1. Operator creates an agent slot via the dashboard; server returns a token.
2. The token is given to the agent at deploy time (env var or config file).
3. Agent sends `Authorization: Bearer <agent_token>` on every request.

## 3. Endpoints

### 3.1 `POST /api/v1/agents/register`

Called by an agent on first startup.

Request:
```json
{
  "agent_kind": "labc2_educational",
  "lab_mode": true,
  "agent_version": "1.0.0",
  "implementation": "bash",
  "hostname": "lab-victim-01",
  "os": "linux",
  "os_version": "Ubuntu 22.04",
  "arch": "x86_64",
  "username": "labuser",
  "pid": 12345,
  "started_at": "2026-05-15T10:30:00Z"
}
```

Required fields: all of the above. `agent_kind` MUST equal
`"labc2_educational"`; servers reject anything else.

Response (200):
```json
{
  "agent_id": "a7f3b2e1-...",
  "check_in_interval_seconds": 30,
  "jitter_percent": 20
}
```

### 3.2 `POST /api/v1/agents/{agent_id}/checkin`

Called every `check_in_interval_seconds` (± jitter).

Request:
```json
{
  "timestamp": "2026-05-15T10:30:30Z",
  "agent_uptime_seconds": 30
}
```

Response (200):
```json
{
  "tasks": [
    {
      "task_id": "b8c4d3f2-...",
      "type": "shell",
      "params": { "command": "whoami" },
      "issued_at": "2026-05-15T10:30:25Z",
      "timeout_seconds": 30
    }
  ]
}
```

The `tasks` array may be empty. Agents MUST process all returned tasks
sequentially before the next check-in.

### 3.3 `POST /api/v1/agents/{agent_id}/results`

Called after each task completes (or fails).

Request:
```json
{
  "task_id": "b8c4d3f2-...",
  "status": "ok",
  "output": "labuser\n",
  "error": null,
  "started_at": "2026-05-15T10:30:30Z",
  "finished_at": "2026-05-15T10:30:30Z"
}
```

`status` is one of: `"ok"`, `"error"`, `"timeout"`.

Response (204 No Content).

### 3.4 `GET /api/v1/agents` (operator)

Lists registered agents and their status.

### 3.5 `POST /api/v1/agents/{agent_id}/tasks` (operator)

Queues a new task for an agent.

Request:
```json
{
  "type": "shell",
  "params": { "command": "uname -a" },
  "timeout_seconds": 30
}
```

Response (201):
```json
{ "task_id": "...", "queued_at": "..." }
```

### 3.6 `GET /api/v1/agents/{agent_id}/tasks` (operator)

Lists all tasks (queued + completed) for an agent, with results.

## 4. Task types

Every agent implementation MUST support all five core task types.

### 4.1 `shell`

```json
{
  "type": "shell",
  "params": { "command": "<string>" }
}
```

Execute the command in the platform's default shell. Capture stdout and
stderr (combined). Return as `output`.

### 4.2 `sysinfo`

```json
{ "type": "sysinfo", "params": {} }
```

Return a JSON-stringified object as `output`:
```json
{
  "hostname": "...", "os": "...", "os_version": "...",
  "arch": "...", "username": "...", "uid": 1000,
  "cwd": "...", "interfaces": [{"name":"eth0","ip":"10.0.0.5"}]
}
```

### 4.3 `download`

Reads a file *from the agent host* and returns it to the server.

```json
{
  "type": "download",
  "params": { "path": "/etc/hostname", "max_bytes": 1048576 }
}
```

`output` is the file content, base64-encoded. If the file exceeds
`max_bytes`, status is `"error"` with `error: "file too large"`.

### 4.4 `upload`

Writes a file *onto the agent host*.

```json
{
  "type": "upload",
  "params": {
    "path": "/tmp/note.txt",
    "content_b64": "aGVsbG8K",
    "mode": "0644"
  }
}
```

### 4.5 `terminate`

```json
{ "type": "terminate", "params": {} }
```

The agent posts a final result (`{"status":"ok"}`) and exits cleanly.
This is a **mandatory** kill switch.

## 5. Error model

Any endpoint can return:

```json
{
  "error": {
    "code": "string_constant",
    "message": "human readable"
  }
}
```

with appropriate HTTP status (400, 401, 403, 404, 409, 500).

Standard error codes:

| Code                       | HTTP | Meaning                                           |
| -------------------------- | ---- | ------------------------------------------------- |
| `invalid_credentials`      | 401  | Bad username/password/TOTP or bad agent token     |
| `not_lab_agent`            | 403  | `agent_kind` != `labc2_educational` on register   |
| `agent_not_found`          | 404  | No agent with that `agent_id`                     |
| `task_not_found`           | 404  | No task with that `task_id`                       |
| `task_type_unsupported`    | 400  | Server received a task type it doesn't know       |
| `validation_failed`        | 400  | Request body failed schema validation             |
| `rate_limited`             | 429  | Too many requests                                 |
| `internal_error`           | 500  | Server bug                                        |

## 6. Versioning

This document is **v1**. The base path is `/api/v1`. Breaking changes go
to `/api/v2`.

## 7. JSON Schemas

Machine-readable schemas for every request/response live in
[schemas/](schemas/). Conformance tests load these to validate every
implementation.
