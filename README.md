# W.A.T.C.H

**W**eb**S**ocket **A**sync **T**elemetry & **C**oncurrent **H**ealth-check

## Highlight

Async SSH polling streams live CPU/memory/disk metrics from multiple Linux servers over a single WebSocket connection.

## Tech Stack

<!-- Badges — for visual display -->
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![asyncssh](https://img.shields.io/badge/asyncssh-4EAA25?style=for-the-badge&logo=ssh&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

<!-- Bullet list — read by the portfolio sync -->
- FastAPI
- Python
- asyncssh
- WebSocket
- React
- JWT
- Docker
- Docker Compose
- GitHub Actions

## Features

- Async SSH polling of CPU/memory/disk stats across multiple Linux servers concurrently (`asyncio.gather`)
- Real-time push to the browser dashboard over WebSocket, no polling on the client side
- JWT-authenticated WebSocket route
- Color-coded threshold states (green/yellow/red) for at-a-glance server health
- Backend accesses monitored servers via a restricted, non-root SSH user — no root/admin required
- Fully separated backend and frontend, containerized independently and wired via Docker Compose

## Getting Started

```bash
git clone https://github.com/subodhadhikari2023/WATCH.git
cd WATCH
```

Setup and run instructions will be added once the Docker/CI scaffolding milestone lands.

---

## Documentation

- [Architecture](docs/architecture.md) — high-level shape of the system
- [System design](docs/system-design.md) — data flow, polling, concurrency, error handling
- [API design](docs/api-design.md) — REST + WebSocket contract, auth, threshold rules
