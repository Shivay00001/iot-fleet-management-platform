# IoT Fleet Management Platform

> A lightweight, high-performance Go service providing the foundational HTTP backbone for an IoT Fleet Management Platform — engineered by **VisionQuantech**.

[![Go Version](https://img.shields.io/badge/Go-1.20-00ADD8?logo=go)](https://go.dev/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-VisionQuantech%20Commercial-red)](LICENSE)

---

## 🚀 Overview

This repository contains the core service binary for the **IoT Fleet Management Platform** — a containerized Go microservice designed to act as the entry point / health-gateway layer for fleet telemetry infrastructure. The service is intentionally minimal, dependency-free (standard library only), and built for fast startup, tiny container footprints, and reliable horizontal scaling behind load balancers or orchestrators such as Kubernetes.

---

## 🏗️ Architecture — How It Works

The service is implemented in a single, clean Go entrypoint (`main.go`) using only the Go standard library (`net/http`, `log`, `fmt`, `time`):

```
┌────────────────────────────────────────────────────────────┐
│                     Client / Load Balancer                  │
└──────────────────────────┬─────────────────────────────────┘
                           │  HTTP GET /
                           ▼
┌────────────────────────────────────────────────────────────┐
│              IoT Fleet Management Service (:8080)           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  net/http DefaultServeMux                            │  │
│  │   └── Route: GET /  →  Operational status handler    │  │
│  │        Responds: "System Operational: <timestamp>"   │  │
│  └──────────────────────────────────────────────────────┘  │
│  Startup: logs "Starting high-performance service on :8080"│
│  Failure: log.Fatal terminates process (container restarts)│
└────────────────────────────────────────────────────────────┘
```

**Key design characteristics:**

| Aspect | Detail |
|---|---|
| **Language / Runtime** | Go 1.20 (`module github.com/Shivay00001/iot-fleet-management-platform`) |
| **Dependencies** | Zero external dependencies — 100% Go standard library |
| **Listening Port** | `8080` (hardcoded) |
| **Endpoint** | `GET /` — returns `System Operational: <current server time>`, making it ideal as a **liveness/readiness probe** for fleet services |
| **Concurrency** | Each request is handled in its own goroutine via `net/http` — naturally high-throughput |
| **Failure Mode** | `log.Fatal` on bind/serve errors → non-zero exit → container orchestrator auto-restart |
| **Container** | Alpine-based multi-stage-ready `Dockerfile` producing a minimal image |

The handler currently serves as the **system heartbeat endpoint**; the architecture is designed so additional fleet routes (device registry, telemetry ingestion, command dispatch) can be registered on the same mux as the platform evolves.

---

## ✨ Features

- ⚡ **High-performance HTTP core** — Go's `net/http` server with goroutine-per-request concurrency.
- 🪶 **Zero-dependency binary** — compiles to a single static executable; no `go mod download` required.
- ❤️ **Built-in health endpoint** — `/` reports live operational status with a server timestamp.
- 🐳 **First-class containerization** — ships with a production-ready `Dockerfile` (Go 1.20 Alpine).
- 🔁 **Crash-safe** — fatal startup errors exit the process cleanly for orchestrator-managed restarts.
- 🔐 **Security-conscious repo hygiene** — `.gitignore` excludes secrets, keys, credentials, and environment files.

---

## 🐳 Running with Docker (Recommended)

The service runs identically on any laptop, server, or cloud VM with Docker installed.

### 1. Build the image

```bash
docker build -t iot-fleet-management-platform .
```

The `Dockerfile` performs the following:
1. Starts from `golang:1.20-alpine`
2. Copies the source into `/app`
3. Compiles the binary via `go build -o app`
4. Launches the service with `CMD ["./app"]`

### 2. Run the container

```bash
docker run -d --name iot-fleet -p 8080:8080 iot-fleet-management-platform
```

### 3. Verify the service

```bash
curl http://localhost:8080/
# → System Operational: 2026-01-01 12:00:00.000000000 +0000 UTC
```

### 4. View logs / stop

```bash
docker logs -f iot-fleet
docker stop iot-fleet && docker rm iot-fleet
```

### Optional: Docker Compose

A `docker-compose.yml` is not included, but this minimal file works out of the box:

```yaml
services:
  iot-fleet:
    build: .
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Then simply run:

```bash
docker-compose up -d --build
```

---

## 🛠️ Running Locally (Without Docker)

**Prerequisite:** Go 1.20+

```bash
# Clone the repository
git clone https://github.com/Shivay00001/iot-fleet-management-platform.git
cd iot-fleet-management-platform

# Build and run
go build -o app .
./app
```

Or run directly:

```bash
go run main.go
```

The service starts on **http://localhost:8080** and logs:

```
Starting high-performance service on :8080
```

---

## 📁 Repository Structure

```
.
├── main.go        # Service entrypoint — HTTP server on :8080
├── go.mod         # Go module definition (Go 1.20, no external deps)
├── Dockerfile     # Container build (golang:1.20-alpine)
├── .gitignore     # Excludes secrets, env files, build artifacts
├── LICENSE        # VisionQuantech Custom Commercial License
└── README.md      # This file
```

---

## 🗺️ Roadmap

The current heartbeat service is the foundation for the full fleet platform:

- [ ] Device registry & provisioning API
- [ ] MQTT / telemetry ingestion pipeline
- [ ] Real-time fleet tracking & geofencing
- [ ] OTA firmware update orchestration
- [ ] Metrics, alerting & observability stack

---

## 📜 License

This project is distributed under the **VisionQuantech Custom Commercial License**:

- ✅ **Free** for personal, educational, and non-earning use.
- 💰 **Revenue share (15–30%)** required for individual/indie projects generating income.
- 🏢 **Commercial license required** for any business or enterprise use — contact **visionquantech@proton.me**.

See [LICENSE](LICENSE) for full terms.

---

<p align="center">
  <b>© 2026 Shivay00001 / VisionQuantech</b><br/>
  Built for scale. Engineered for fleets.
</p>