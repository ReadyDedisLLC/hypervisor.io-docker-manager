# Hypervisor Docker Apps

Community-maintained catalog of one-click Docker applications for the [Hypervisor](https://hypervisor.io) control panel.

Users can deploy these apps directly onto their virtual machines from the Docker Manager tab — no SSH required.

---

## How It Works

Each app in this repository is a self-contained directory with a Docker Compose file, a metadata file (`app.json`), and an icon. When a user clicks "Deploy" in the Hypervisor panel, the platform:

1. Pulls the app definition from this repository
2. Presents a configuration form (environment variables, port mappings)
3. Injects a deploy script into the VM via the QEMU guest agent
4. Runs `docker compose up` inside the VM with the user's configuration
5. Reports deployment status back to the control panel

The VM must have Docker Engine installed (enabled during instance creation).

---

## Repository Structure

```
apps/
├── grafana/
│   ├── app.json              # App metadata and configuration schema
│   ├── docker-compose.yml    # Compose file deployed to the VM
│   └── icon.png              # 128x128 app icon
├── wordpress/
│   ├── app.json
│   ├── docker-compose.yml
│   └── icon.png
└── ...
```

---

## Contributing an App

### 1. Fork & Clone

```bash
git clone https://github.com/<your-fork>/docker-apps.git
cd docker-apps
```

### 2. Create the App Directory

```bash
mkdir apps/your-app-name
```

Use a lowercase slug with hyphens. This becomes the app's unique identifier.

### 3. Write `app.json`

```json
{
  "name": "Your App",
  "slug": "your-app-name",
  "description": "A short description of what this app does.",
  "icon": "icon.png",
  "category": "category-name",
  "version": "latest",
  "website": "https://example.com",
  "supported_distros": [
    "ubuntu-22.04",
    "ubuntu-24.04",
    "debian-12",
    "rocky-9",
    "almalinux-9"
  ],
  "min_ram_mb": 512,
  "min_storage_gb": 5,
  "env": [],
  "ports": []
}
```

### 4. Write `docker-compose.yml`

```yaml
services:
  your-app:
    image: your-app:latest
    restart: unless-stopped
    ports:
      - "${HOST_PORT:-8080}:8080"
    env_file:
      - .env
    volumes:
      - app_data:/data

volumes:
  app_data:
```

### 5. Add an Icon

Place a **128x128 PNG** named `icon.png` in the app directory. Use the official logo of the application.

### 6. Submit a Pull Request

```bash
git checkout -b add-your-app-name
git add apps/your-app-name/
git commit -m "Add your-app-name"
git push origin add-your-app-name
```

Open a PR against `main` with a brief description of the app and a link to its official documentation.

---

## `app.json` Reference

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Display name shown in the catalog |
| `slug` | string | Yes | Unique identifier (lowercase, hyphens only). Must match the directory name |
| `description` | string | Yes | Short description (1-2 sentences) |
| `icon` | string | Yes | Filename of the icon (relative to app directory) |
| `category` | string | Yes | Category for catalog filtering (see [Categories](#categories)) |
| `version` | string | Yes | Default image tag. Use `"latest"` unless a specific version is required |
| `website` | string | No | Link to the app's homepage or docs |
| `supported_distros` | array | Yes | List of supported OS distro slugs |
| `min_ram_mb` | integer | Yes | Minimum RAM in MB required to run the app |
| `min_storage_gb` | integer | Yes | Minimum disk space in GB |
| `env` | array | Yes | Environment variable definitions (can be empty `[]`) |
| `ports` | array | Yes | Port mapping definitions (can be empty `[]`) |

### Environment Variables (`env[]`)

Each entry defines a configurable environment variable shown to the user before deployment.

| Field | Type | Required | Description |
|---|---|---|---|
| `key` | string | Yes | Environment variable name (e.g. `MYSQL_ROOT_PASSWORD`) |
| `label` | string | Yes | Human-readable label shown in the form |
| `default` | string | No | Default value. Leave empty `""` for user-provided values |
| `required` | boolean | Yes | Whether the field must be filled before deployment |
| `type` | string | Yes | Input type: `"text"`, `"password"`, `"number"`, `"select"` |
| `generate` | boolean | No | If `true` and type is `"password"`, a random secure password is auto-generated |
| `options` | array | No | For `type: "select"` only. List of `{"label": "...", "value": "..."}` objects |
| `description` | string | No | Help text shown below the input field |

### Port Mappings (`ports[]`)

Each entry defines a port the container exposes, with an optional user-configurable host port.

| Field | Type | Required | Description |
|---|---|---|---|
| `container_port` | integer | Yes | The port inside the container |
| `default_host_port` | integer | Yes | Default port on the host VM |
| `protocol` | string | Yes | `"tcp"` or `"udp"` |
| `label` | string | Yes | Description shown to the user (e.g. `"Web UI"`, `"Database"`) |
| `configurable` | boolean | Yes | Whether the user can change the host port |

### Categories

Use one of these categories. If your app doesn't fit, suggest a new one in your PR.

| Category | Examples |
|---|---|
| `analytics` | Plausible, Umami, Matomo |
| `automation` | n8n, Huginn, Node-RED |
| `cms` | WordPress, Ghost, Strapi |
| `communication` | Rocket.Chat, Mattermost |
| `databases` | PostgreSQL, MySQL, Redis, MongoDB |
| `development` | Gitea, GitLab, code-server |
| `file-storage` | Nextcloud, Seafile, MinIO |
| `gaming` | Minecraft, Valheim, Palworld |
| `media` | Jellyfin, Plex, Navidrome |
| `monitoring` | Grafana, Uptime Kuma, Prometheus |
| `networking` | Pi-hole, AdGuard Home, WireGuard |
| `productivity` | Vikunja, Bookstack, Wiki.js |
| `security` | Vaultwarden, Authelia, CrowdSec |

### Supported Distros

Use these slugs. Only list distros you have tested against.

```
ubuntu-22.04, ubuntu-24.04
debian-11, debian-12
rocky-8, rocky-9
almalinux-8, almalinux-9
centos-stream-9
```

---

## Full Example: Uptime Kuma

**`apps/uptime-kuma/app.json`**

```json
{
  "name": "Uptime Kuma",
  "slug": "uptime-kuma",
  "description": "Self-hosted monitoring tool for tracking uptime of websites, APIs, and services.",
  "icon": "icon.png",
  "category": "monitoring",
  "version": "1",
  "website": "https://github.com/louislam/uptime-kuma",
  "supported_distros": [
    "ubuntu-22.04",
    "ubuntu-24.04",
    "debian-11",
    "debian-12",
    "rocky-9",
    "almalinux-9"
  ],
  "min_ram_mb": 256,
  "min_storage_gb": 2,
  "env": [],
  "ports": [
    {
      "container_port": 3001,
      "default_host_port": 3001,
      "protocol": "tcp",
      "label": "Web UI",
      "configurable": true
    }
  ]
}
```

**`apps/uptime-kuma/docker-compose.yml`**

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    restart: unless-stopped
    ports:
      - "${HOST_PORT_3001:-3001}:3001"
    volumes:
      - uptime_kuma_data:/app/data

volumes:
  uptime_kuma_data:
```

---

## Guidelines

- **One service per app.** If your app requires multiple containers (e.g. WordPress + MySQL), include them all in a single `docker-compose.yml`.
- **Use named volumes.** Never bind-mount to host paths. Named volumes are portable and work across distros.
- **Use `restart: unless-stopped`.** Apps should survive VM reboots.
- **Use `.env` file for secrets.** Reference variables with `${VAR_NAME}` syntax in your compose file. The platform writes the `.env` file automatically from the user's form input.
- **Pin major versions.** Use `image: app:1` or `image: app:3.0` rather than `image: app:latest` when the app has breaking changes between majors.
- **Keep it minimal.** Don't add reverse proxies, SSL terminators, or monitoring sidecars. Users manage those separately.
- **Test your compose file.** Run `docker compose up -d` locally and verify the app starts cleanly before submitting.

## Port Mapping Convention

Host ports in `docker-compose.yml` should use the environment variable format `${HOST_PORT_<container_port>:-<default>}`:

```yaml
ports:
  - "${HOST_PORT_3000:-3000}:3000"   # Maps to ports[] entry with container_port: 3000
  - "${HOST_PORT_5432:-5432}:5432"   # Maps to ports[] entry with container_port: 5432
```

The platform sets these environment variables based on the user's port configuration.

---

## License

App definitions in this repository are licensed under [MIT](LICENSE). Individual Docker images are subject to their own licenses.
