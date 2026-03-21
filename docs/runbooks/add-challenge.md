# Adding a New Challenge

## Overview

This runbook describes the process of adding and deploying a new challenge to the CTF platform.

---

## Prerequisites

- Challenge Docker image is built or Dockerfile is ready
- Challenge runs correctly in local environment
- Port and service details are known

---

## Steps

### 1. Prepare Challenge Locally

- Build the Docker image:
  ```bash
  docker build -t challenge-name .
```

* Run locally to verify:

  ```bash
  docker run -p 5000:5000 challenge-name
  ```

* Verify:

  * Service is accessible
  * Challenge works as intended
  * Flag is retrievable

---

### 2. Push Image (if using registry)

* Tag image:

  ```bash
  docker tag challenge-name registry/challenge-name
  ```

* Push to registry:

  ```bash
  docker push registry/challenge-name
  ```

---

### 3. Deploy Container via Portainer

* Open Portainer UI
* Create or start container:

  * Use image or build config
  * Assign port
  * Attach to correct Docker network

---

### 4. Configure Traefik Routing

* Add appropriate labels to container:

  * Enable Traefik
  * Define router rule (host/path)
  * Set service port

* Example:

  ```
  traefik.enable=true
  traefik.http.routers.challenge.rule=Host(`challenge.domain`)
  ```

---

### 5. Verify Deployment

* Access challenge via browser
* Confirm:

  * Route works
  * No errors
  * Challenge behaves correctly

---

### 6. Add Monitoring

* Add endpoint to Uptime Kuma
* Verify status is “up”

---

## Verification Checklist

* [ ] Container is running
* [ ] Challenge accessible via Traefik route
* [ ] No errors in logs
* [ ] Uptime Kuma shows healthy status

---

## Common Issues

* Port mismatch between container and Traefik
* Missing or incorrect labels
* Container not attached to correct network
* Service not listening on expected port

---

## Notes

* Always test locally before deployment
* Avoid port conflicts with existing challenges
* Keep naming consistent across containers and routes

