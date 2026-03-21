# Deploy CTF Runbook

## Overview

This runbook covers the end-to-end process for deploying all challenge containers
for a CTF event — from pre-event preparation through event-day monitoring to
post-event teardown.

---

## Pre-Event Checklist

Complete these steps before the event starts.

### Infrastructure

- [ ] VPS is accessible via SSH
- [ ] Traefik is running and healthy
- [ ] Portainer is accessible
- [ ] Uptime Kuma is running
- [ ] DNS records for all challenge subdomains are configured and propagated

### Challenges

- [ ] All challenge Docker images build successfully (`docker build`)
- [ ] All challenges run correctly in local testing (`docker compose up`)
- [ ] Each challenge has a valid, fully-filled `challenge.yaml` (including `description`)
- [ ] Flags are confirmed correct for all challenges
- [ ] All `docker-compose.yml` files have correct Traefik labels and domain names

---

## Deployment Steps

### 1. SSH into the VPS

```bash
ssh <user>@<vps-ip>
```

### 2. Pull latest challenge code

```bash
git pull
```

### 3. Build and start challenge containers

For each challenge, from its directory:

```bash
docker compose up --build -d
```

Or to deploy all challenges at once from the repo root (if using a top-level
compose file):

```bash
docker compose up --build -d
```

### 4. Verify containers are running

```bash
docker ps
```

Confirm each challenge container appears with status `Up`.

### 5. Check Traefik routing

1. Open the Traefik dashboard (if enabled)
2. Confirm each challenge router appears under `HTTP Routers`
3. Verify the `Host(...)` rule matches the intended subdomain

If a challenge does not appear in Traefik:

```bash
# Restart Traefik to force label re-discovery
docker restart traefik
```

### 6. Verify challenge accessibility

For each challenge, confirm it is reachable via its public URL:

```bash
curl -I https://<challenge-name>.<domain>
```

Expected: `200 OK` (or appropriate response for the challenge type).

### 7. Register endpoints in Uptime Kuma

For each newly deployed challenge:

1. Open Uptime Kuma
2. Add a new monitor:
   - **Type:** HTTP(s) or TCP depending on challenge type
   - **URL / Address:** the challenge's public URL or host:port
   - **Friendly Name:** challenge name
3. Confirm the monitor shows `Up` within the check interval

---

## Event-Day Monitoring

During the event, monitor the following:

- [ ] Uptime Kuma dashboard — all challenges show `Up`
- [ ] Container logs for errors:
  ```bash
  docker logs <container-id> --follow
  ```
- [ ] Traefik logs for routing errors:
  ```bash
  docker logs traefik --follow
  ```
- [ ] VPS resource usage (CPU, memory, disk):
  ```bash
  htop
  df -h
  ```

### Restarting a broken challenge

```bash
docker compose restart <service-name>
```

Or force a full rebuild if the image is suspect:

```bash
docker compose up --build -d <service-name>
```

---

## Known Issues

- **Traefik does not detect new containers immediately** — restart Traefik if a
  challenge route is missing after deployment.
- **Portainer UI may show stale state** — refresh or restart Portainer if
  container status appears incorrect.
- **Port conflicts** — if a challenge fails to start, check that its host port
  is not already in use: `docker ps` or `ss -tlnp`.

---

## Post-Event Teardown

After the event has concluded:

### 1. Stop and remove challenge containers

```bash
docker compose down
```

### 2. Remove challenge images (optional, to free disk space)

```bash
docker image prune -a
```

### 3. Disable challenge monitors in Uptime Kuma

Mark each challenge monitor as paused or delete it.

### 4. Archive challenge containers

Confirm all challenge source code, flags, and solve scripts are committed to
`ctf-challenges` before removing anything locally.

---

## Verification Checklist (Post-Deployment)

- [ ] All challenge containers are running (`docker ps`)
- [ ] All challenge URLs return expected responses
- [ ] Traefik routes are correctly configured
- [ ] No `5xx` errors in Traefik logs
- [ ] Uptime Kuma shows all services healthy
- [ ] No unnecessary ports are exposed on the VPS
