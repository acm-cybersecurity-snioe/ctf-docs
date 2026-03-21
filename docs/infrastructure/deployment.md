# Deployment Process

## Overview

This document describes the current manual deployment process for the CTF infrastructure and challenges.

Deployment is currently performed using Portainer and manual configuration of services.

---

## VPS Hardening (Initial Setup)

This step is performed when provisioning a new VPS or rebuilding infrastructure. It is not required for every CTF deployment.

### 1. Secure SSH Access
- Disable password authentication
- Enable key-based authentication
- Disable root login
- Optionally change default SSH port (non-essential, reduces noise)

### 2. Configure Firewall (UFW)
- Allow only required ports:
  - SSH
  - HTTP (80)
  - HTTPS (443)
- Deny all other incoming traffic

### 3. Install and Configure Fail2Ban
- Protect SSH from brute-force attacks
- Verify jail configuration is active

### 4. Enable Automatic Security Updates
- Configure unattended upgrades
- Ensure system receives security patches regularly

### 5. Privilege Escalation Audit
- Run linpeas to identify potential privilege escalation vectors
- Review and mitigate any critical findings

---

## Prerequisites

- Access to VPS (SSH)
- Access to Portainer UI
- All required Docker images built or available

---

## Deployment Steps

### 1. Access Server
- SSH into the VPS hosting the infrastructure

### 2. Open Portainer
- Navigate to Portainer web interface
- Select the appropriate environment

### 3. Deploy / Update Core Infrastructure
- Create or update stack using Docker Compose (if applicable)
- Ensure core services are running:
  - Traefik
  - Uptime Kuma

### 4. Deploy Challenge Containers
- Start individual challenge containers
- Verify containers are running

### 5. Configure Routing (Traefik)
- Ensure each challenge is properly routed
- Verify labels or routing rules

### 6. Verify Services
- Access each challenge via browser
- Confirm correct responses

### 7. Monitor via Uptime Kuma
- Check that all services are marked “up”
- Investigate any failures

---

## Verification Checklist

- [ ] Traefik is running
- [ ] All challenge containers are running
- [ ] Routes are accessible
- [ ] No 5xx errors
- [ ] Uptime Kuma shows all services healthy

---

## Known Issues

- Traefik may not detect new containers immediately → restart may be required
- Portainer UI can sometimes show stale state
- Manual routing mistakes can break challenge access

---

## Current Limitations

- Fully manual deployment
- No automated validation of challenges
- No rollback mechanism

---

## Notes

- VPS hardening should be completed before exposing services publicly
- Security audits should be repeated periodically or after major infrastructure changes

