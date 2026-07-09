# Architecture

## Overview

Three logical pieces, connected in one direction only:

```
[Monitored Server 1] \
[Monitored Server 2]  --- SSH (asyncssh) --->  [FastAPI Backend]  --- WebSocket --->  [React Frontend]
[Monitored Server N] /
```

1. **Monitored servers** — plain Linux boxes, accessed via SSH with a restricted,
   non-root `monitor` user. Reading CPU/memory/disk stats never requires root.

2. **Backend (FastAPI)** — the only piece that talks to monitored servers. Uses
   `asyncssh` (native async, not `paramiko` + a thread pool) to poll all servers
   concurrently, and pushes results to the frontend over a WebSocket.

3. **Frontend (React)** — talks only to the backend, never directly to the
   monitored servers.

## Why this shape

- **Restricted SSH user** — the backend's SSH credentials can only read system
  stats, not administer the box. If those credentials leak, the blast radius
  is read-only telemetry, not server control.
- **Native async SSH** — `asyncssh` lets one backend process hold many
  concurrent SSH sessions on a single event loop, instead of a thread per
  server. This is central to how the backend scales to N servers.
- **Single trust boundary** — the frontend never touches monitored servers
  directly. The backend is the only component with SSH credentials, and the
  only component the frontend needs to trust.

## Project layout

```
backend/    FastAPI service, asyncssh polling, WebSocket push
frontend/   React dashboard
docs/       Design docs (this file, system design, API contract)
```

`backend/` and `frontend/` share no code — they communicate only over
HTTP/WebSocket, per `docs/api-design.md` (once written).
