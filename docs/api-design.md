# API Design

The formal contract between backend and frontend. Written after
`docs/system-design.md` and the Figma/Excalidraw mockups on purpose — the
mockups are what pinned down the exact fields below (friendly `name` vs raw
IP, a `lastUpdated` timestamp, an explicit `reachable` flag for the
unreachable card state).

Field naming is **camelCase** across every JSON payload, even though the
backend is Python — the frontend is the sole consumer of this contract, so it
gets to set the convention (`alias_generator` in Pydantic models handles the
snake_case ↔ camelCase translation server-side).

There is no `GET /servers` REST endpoint — the WebSocket snapshot below *is*
the server list plus stats, including the first one a client receives right
after connecting. There's also no logout endpoint: JWTs are stateless, so
"logout" is purely client-side (drop the token, redirect to login).

## REST: `POST /auth/login`

**Request**
```json
{
  "username": "admin",
  "password": "hunter2"
}
```

**Response 200**
```json
{
  "token": "<JWT>",
  "expiresAt": "2026-07-10T11:15:32Z"
}
```

**Response 401**
```json
{
  "error": "invalid_credentials",
  "message": "Invalid username or password"
}
```

JWT claims: `sub` (username), `iat`, `exp`. `expiresAt` in the response body
is just `exp` rendered as ISO 8601 UTC, so the frontend doesn't need to
decode the token to know when to expect a session to end.

## WebSocket: `/ws/servers`

### Handshake

1. Client opens the WebSocket connection (no token in the URL).
2. First message, client → server:
   ```json
   { "type": "auth", "token": "<JWT>" }
   ```
3. Server validates the token:
   - **Invalid, missing, or malformed** → server sends an `error` message
     (below) and closes with code **4401**.
   - **Valid** → server records that connection's `exp`, and begins
     streaming `snapshot` messages on the regular 5-second tick from
     `docs/system-design.md`.

### Mid-session expiry

Every tick, alongside polling servers, the backend checks each open
connection's stored `exp`. If a connection's token has expired since the
handshake:
- Server sends an `error` message (below) and closes with code **4402**.
- The connection is dropped — there is no re-auth-in-place; the client must
  log in again and open a new connection.

### Close codes

| Code | Meaning | Frontend behavior |
|---|---|---|
| 4401 | `auth_failed` — bad/missing/invalid token at handshake | Redirect to login. Do **not** enter the reconnect-backoff loop — a retry with the same bad token will just fail again. |
| 4402 | `token_expired` — token was valid at connect, expired mid-session | Redirect to login (session ended, not a network blip). |
| *(any other)* | Ordinary disconnect (network blip, server restart, etc.) | Normal reconnect-with-backoff from `docs/system-design.md` applies. |

This is the refinement `docs/system-design.md`'s reconnection section
needed: "reconnect" there assumed every disconnect is transient. 4401/4402
are the exception — reconnecting with the same expired/invalid token would
just be rejected again, so those two codes route to login instead of the
backoff loop.

### Server → client messages

**`error`** (sent immediately before closing on 4401/4402)
```json
{
  "type": "error",
  "code": "token_expired",
  "message": "Session expired, please log in again."
}
```

**`snapshot`** (every 5 seconds, once authenticated)
```json
{
  "type": "snapshot",
  "servers": [
    {
      "name": "prod-web-01",
      "ip": "10.0.4.11",
      "reachable": true,
      "cpuPercent": 42.0,
      "memPercent": 61.0,
      "diskPercent": 55.0,
      "lastUpdated": "2026-07-10T10:15:32Z"
    },
    {
      "name": "prod-db-01",
      "ip": "10.0.5.10",
      "reachable": false,
      "cpuPercent": null,
      "memPercent": null,
      "diskPercent": null,
      "lastUpdated": "2026-07-10T10:14:48Z"
    }
  ]
}
```

- `reachable: false` is what the backend's 3-consecutive-failures rule
  (`docs/system-design.md`) surfaces to the frontend. When `false`, the
  metric fields are `null` — there's nothing to report — and `lastUpdated`
  is the last time a poll actually succeeded (so the frontend can show how
  stale the data is, matching the "47s ago" unreachable-card mockup).
- `cpuPercent`/`memPercent`/`diskPercent` are raw numbers, not colors —
  per `docs/system-design.md`, the backend stays a dumb data pipe and the
  frontend computes threshold color itself (below).
- No per-server `id`: `name` does double duty as display name and identifier,
  since servers are static backend config, not something added/edited
  through the UI (out of scope per the brief).

## Threshold color rule (frontend-computed)

Nothing had defined actual cutoffs yet — the mockups only used illustrative
numbers. Proposed default, applied identically to CPU/MEM/DISK:

| Range | Color |
|---|---|
| `< 70%` | Green (normal) |
| `70% – 90%` | Yellow (warning) |
| `> 90%` | Red (critical) |

A card's overall color is the **worst** of its three metrics (per the
whole-card-highlight decision from the wireframe stage). These numbers are a
starting point, not a locked contract — easy to retune once there's a real
server generating real load data.
