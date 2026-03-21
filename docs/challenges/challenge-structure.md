# Challenge Structure Standard

## Overview

All challenges must follow a standardized structure to ensure consistency,
reproducibility, and ease of deployment.

---

## Repository Layout

Challenges are organized by **year**, then **category**, then **difficulty**:

```
ctf-challenges/
  templates/                  # copy-paste scaffolds — one per category
  <year>/                     # e.g. 2026/
    <category>/               # e.g. web/
      <difficulty>/           # e.g. easy/
        <challenge-name>/     # e.g. sqli-1/
          Dockerfile
          docker-compose.yml
          challenge.yaml
          src/
          solve/
```

### Example

```
ctf-challenges/
  2026/
    web/
      easy/
        sqli-1/
    pwn/
      medium/
        buffer-overflow-1/
  2027/
    crypto/
      hard/
        rsa-3/
```

---

## Valid Categories

| Category    | Description                             |
|-------------|-----------------------------------------|
| `web`       | Web application vulnerabilities         |
| `pwn`       | Binary exploitation / memory corruption |
| `crypto`    | Cryptographic weaknesses                |
| `forensics` | File, memory, and network analysis      |
| `stego`     | Steganography and hidden data           |
| `rev`       | Reverse engineering                     |
| `misc`      | Challenges that don't fit elsewhere     |

## Valid Difficulties

| Difficulty | Description                              |
|------------|------------------------------------------|
| `easy`     | Introductory — suitable for beginners    |
| `medium`   | Intermediate — requires some experience  |
| `hard`     | Advanced — significant skill required    |

---

## Per-Challenge Required Files

Every challenge directory must contain exactly these files and directories:

```
challenge-name/
  Dockerfile          # builds the challenge image
  docker-compose.yml  # defines how the challenge runs
  challenge.yaml      # metadata
  src/                # challenge source code
  solve/              # intended solution and exploit scripts
```

---

## Required Files

### `Dockerfile`

Defines how the challenge container is built.

Requirements:

- Use a minimal base image (e.g. `python:3.12-slim`, `node:20-alpine`)
- Pin all dependency versions for reproducibility
- Expose exactly one port per challenge (default: `5000`)
- Copy only required files into the image
- Challenges must be fully self-contained (no external runtime dependencies)

---

### `docker-compose.yml`

Defines how the challenge runs.

Requirements:

- Must define the service clearly using the challenge name
- Must include Traefik labels for production routing
- Use `restart: unless-stopped` for all challenge services
- Avoid hardcoding secrets in compose files

```yaml
services:
  challenge-name:
    build: .
    ports:
      - "5000:5000"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.<challenge-name>.rule=Host(`<challenge-name>.<domain>`)"
    restart: unless-stopped
```

---

### `challenge.yaml`

Metadata file for the challenge.

```yaml
name: sqli-1
category: web
difficulty: easy
author: your-name
description: "A short description of the challenge scenario."
flag: CTF{example_flag}
flag_env: FLAG          # do not change — override only if strictly required (heavily discouraged)
port: 5000
```

| Field         | Required | Description                                                             |
|---------------|----------|-------------------------------------------------------------------------|
| `name`        | yes      | Lowercase, hyphen-separated identifier                                  |
| `category`    | yes      | One of the valid categories listed above                                |
| `difficulty`  | yes      | `easy`, `medium`, or `hard`                                             |
| `author`      | yes      | Name or handle of the challenge author                                  |
| `description` | yes      | Short description of the challenge scenario                             |
| `flag`        | yes      | The flag in `CTF{...}` format                                           |
| `flag_env`    | yes      | Environment variable name the challenge reads the flag from — must be `FLAG` in virtually all cases; overriding is heavily discouraged |
| `port`        | yes      | Port the service listens on inside the container                        |

---

### `src/`

Contains all challenge source code — everything needed to build and run the
challenge service. The `Dockerfile` should copy from this directory.

---

### `solve/`

Contains:

- A written explanation of the vulnerability and intended solution path
- Exploit scripts or solve scripts (if applicable)
- Expected output demonstrating a successful solve

The `solve/` directory is for internal reference only — never expose it to players.

---

## Flag Injection

Flags must be injected into containers via the `FLAG` environment variable at
runtime. **Never hardcode the flag in the image** — hardcoded flags can be
extracted by inspecting image layers.

### Standard pattern

Pass `FLAG` through `docker-compose.yml`:

```yaml
environment:
  - FLAG=${FLAG}   # resolved from a .env file or Portainer secrets
```

Or when running manually:

```bash
docker run -e FLAG='CTF{your_flag}' -p 5000:5000 <challenge-name>
```

Inside the challenge code, read the flag from the environment:

| Language | Example |
|----------|---------|
| Python   | `os.environ.get("FLAG")` |
| Node.js  | `process.env.FLAG` |
| Bash     | `echo $FLAG` |
| C        | `getenv("FLAG")` |

For compiled binaries or artifact-based challenges (forensics, stego, rev),
use an entrypoint script to embed `FLAG` into the relevant file or location at
container start — not at build time.

### The `flag_env` field

The `flag_env` field in `challenge.yaml` documents which environment variable
the challenge reads the flag from. The default and expected value is `FLAG`.

**Overriding `flag_env` is heavily discouraged.** It exists only for the narrow
edge case where a challenge wraps a third-party service that already expects a
differently-named variable and cannot be modified. If you find yourself wanting
to change it for any other reason, reconsider the challenge design instead.

### Local development

Create a `.env` file in the challenge directory (never commit it):

```bash
FLAG=CTF{test_flag_local}
```

Docker Compose resolves it automatically:

```bash
docker compose up --build
```

---

## Using Templates

The `templates/` directory at the root of `ctf-challenges/` contains a
ready-to-copy scaffold for each category. To start a new challenge:

1. Copy the relevant template:

   ```bash
   cp -r templates/web 2026/web/easy/my-challenge
   ```

2. Fill in `challenge.yaml` with the correct metadata.
3. Add challenge source code to `src/`.
4. Add the intended solution to `solve/`.
5. Write the `Dockerfile` and `docker-compose.yml` for your stack.

---

## Naming Conventions

| Thing                   | Convention       | Example                        |
|-------------------------|------------------|--------------------------------|
| Challenge directory     | lowercase-hyphen | `sqli-1`, `buffer-overflow-2`  |
| Docker image/service    | lowercase-hyphen | `web-sqli-1`                   |
| `challenge.yaml` `name` | lowercase-hyphen | `sqli-1`                       |
| Flag format             | `CTF{...}`       | `CTF{example_flag_here}`       |
| Traefik router name     | lowercase-hyphen | `traefik.http.routers.sqli-1`  |

---

## Port Guidelines

- All challenges use internal port `5000` by default
- Avoid port conflicts between challenges running simultaneously on the same host
- The internal port should always be `5000`; the host port can vary

---

## Validation Checklist

Before submitting a challenge:

- [ ] `docker build` succeeds with no errors
- [ ] Container starts and service is reachable on port 5000
- [ ] Challenge behaves as intended
- [ ] Flag is retrievable via the intended solution in `solve/`
- [ ] No unnecessary services or open ports inside the container
- [ ] `challenge.yaml` is fully filled out (including `description`)

---

## Notes

- Challenges must be self-contained — no external runtime dependencies
- Avoid storing secrets in the image beyond the flag itself
- Ensure reproducibility across environments
