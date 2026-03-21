# Architecture Overview

## High-Level Flow

User → Traefik → Challenge Containers

All services are hosted on a single VPS and exposed via HTTP routing.

---

## Components

### Traefik (Reverse Proxy)
Traefik handles all incoming HTTP requests and routes them to the correct challenge container.

- Uses Docker labels for routing
- Central entry point for all traffic
- Can be extended with rate limiting and middleware

---

### Challenge Containers
Each challenge runs inside its own Docker container.

- Isolated environment per challenge
- Exposes a service (web, TCP, etc.)
- Routed via Traefik

---

### Portainer
Portainer is used for managing Docker containers and stacks.

- Provides a UI for deployment and monitoring
- Not the source of truth (manual operations currently)

---

### Uptime Kuma
Used for monitoring service availability.

- Tracks uptime of challenge endpoints
- Helps detect failures during events

---

## Networking Model

- All containers run on the same Docker host
- Traefik connects to containers via Docker network
- External traffic flows only through Traefik

---

## Current Limitations

- Deployment is manual (no CI/CD)
- No centralized logging
- Limited observability into container behavior

---

## Future Improvements (Planned)

- Infrastructure as Code (Docker Compose)
- Centralized logging (Loki stack)
- Metrics and monitoring (Prometheus + Grafana)
- Automated deployments
