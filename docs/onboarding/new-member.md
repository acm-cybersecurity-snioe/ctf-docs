# New Member Guide

## Overview

This guide helps new members understand the CTF platform and get started with contributing.

---

## What You Need to Know

The platform consists of three main parts:

- `ctf-docs` → documentation (you are here)
- `ctf-infra` → infrastructure setup (Traefik, Docker, etc.)
- `ctf-challenges` → challenge implementations

---

## Prerequisites

You should be familiar with:

- Basic Linux commands
- Git and GitHub workflow
- Docker (basic usage)

---

## Local Setup

### 1. Clone Repositories

```bash
git clone <ctf-docs>
git clone <ctf-challenges>
git clone <ctf-infra>
````

---

### 2. Install Required Tools

* Docker
* Python (for MkDocs if working on docs)
* Git

---

### 3. Run a Sample Challenge

Inside a challenge directory:

```bash
docker build -t test-challenge .
docker run -p 5000:5000 test-challenge
```

Then open:

```
http://localhost:5000
```

---

## How to Contribute

### Writing a Challenge

1. Follow the challenge structure standard
2. Build and test locally
3. Deploy using the runbook
4. Verify everything works

---

### Working on Infrastructure

1. Understand current architecture
2. Make changes carefully
3. Test before deploying

---

### Improving Documentation

1. Update relevant docs
2. Keep content clear and actionable
3. Avoid unnecessary theory

---

## Recommended Reading

* Architecture Overview
* Deployment Process
* Add Challenge Runbook

---

## Guidelines

* Keep things simple and reproducible
* Document what you change
* Test before pushing changes

---

## Getting Help

* Ask in team group/chat
* Check documentation first
* Review existing challenges for reference
