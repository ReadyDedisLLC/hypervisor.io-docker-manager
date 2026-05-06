# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a community-maintained catalog of ~150 one-click Docker applications for the [Hypervisor](https://hypervisor.io) control panel. The Hypervisor platform reads app definitions from this repo, presents a configuration form to users, and runs `docker compose up` inside VMs via the QEMU guest agent.

No build, lint, or test infrastructure. This is a pure data/content repository.

## Repository Structure

```
apps/
├── index.json              # Full catalog array — every app's app.json content
├── <app-slug>/
│   ├── app.json            # Metadata, env vars, ports, requirements
│   ├── docker-compose.yml  # Deployed to VM; env_file: .env for secrets
│   └── icon.png            # 128x128 PNG
```

`apps/index.json` must be updated whenever an app is added, removed, or modified. It is a JSON array where each entry is the complete `app.json` content for that app.

## Common Operations

**Validate an app.json is well-formed:**
```bash
jq . apps/<slug>/app.json > /dev/null && echo "valid" || echo "invalid"
```

**Count apps in catalog:**
```bash
jq 'length' apps/index.json
```

**List all app slugs:**
```bash
ls -d apps/*/ | sed 's|apps/||;s|/||'
```

**Check which apps define password env vars with generate:true (auto-generated secrets):**
```bash
jq -r '[.[] | select(.env[]?.generate == true) | .slug]' apps/index.json
```

**Check which apps have no env vars at all:**
```bash
jq -r '.[] | select(.env | length == 0) | .slug' apps/index.json
```

**Find apps using a specific env var pattern:**
```bash
grep -r "key.*MYSQL" apps/*/app.json
```

## Key Conventions

### docker-compose.yml

- `env_file: .env` for all secrets (platform writes the `.env` file from user form input)
- Ports: `${HOST_PORT_<container_port>:-<default>}:<container_port>` — `container_port` must match a `ports[]` entry in app.json
- `restart: unless-stopped` on all services
- Named volumes only, never bind mounts to host paths
- Multi-service apps: include all containers (app + DB + cache) in one compose file
- Pin major versions when app has breaking changes between majors: `image: app:1` not `image: app:latest`

### app.json

- `slug` must match the directory name exactly (lowercase, hyphens only)
- `version`: use `"latest"` unless a specific major version is required
- Env var types: `"text"`, `"password"`, `"number"`, `"select"`
- Password vars with `"generate": true` get auto-generated random passwords
- `supported_distros`: only list distros the app has been tested against. Valid slugs: `ubuntu-22.04`, `ubuntu-24.04`, `debian-11`, `debian-12`, `rocky-8`, `rocky-9`, `almalinux-8`, `almalinux-9`, `centos-stream-9`
- Categories: `ai`, `analytics`, `automation`, `cms`, `communication`, `databases`, `development`, `file-storage`, `gaming`, `media`, `monitoring`, `networking`, `productivity`, `security`

### Recent Work Patterns

Recent commits focused on:
- Adding missing credential/secret env vars to apps that were deployed without them
- Fixing hardcoded secrets — should always use env vars via `.env`
- Mass additions of new apps (65 added in a single commit)
- Ensuring `env[]` entries in app.json match the variables referenced in docker-compose.yml

When adding or modifying an app, verify that every environment variable referenced in docker-compose.yml (via `${VAR_NAME}`) has a corresponding entry in app.json `env[]`, and vice versa. Password-typed fields that require credentials should have `"generate": true` or valid defaults.
