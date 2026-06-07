# Nginx Proxy Manager - Docker Stack

Self-hosted reverse proxy management with an admin UI, Let's Encrypt
certificate handling, and persistent MariaDB storage.

This repository ships a sanitized Docker Compose stack designed to be deployed
by a stack manager (Komodo, Portainer, Dockge, etc.) or directly with
`docker compose`.

## Requirements

- Docker Engine 24+ with Compose v2
- Free ports `80/tcp`, `443/tcp`, and `81/tcp` on the host
- Strong database passwords supplied through `.env` or your stack manager

## Quick Start

```bash
git clone https://github.com/oehrn/Nginx.git
cd Nginx
cp .env.example .env
# Edit .env and replace both change_me values with strong random passwords.
docker compose up -d
```

The admin UI is reachable at `http://<host-ip>:81`.

## Initial Login

Use the upstream default login only for the first sign-in:

| Field | Value |
|---|---|
| Email | `admin@example.com` |
| Password | `changeme` |

Change the email and password immediately after the first login.

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `NPM_CONTAINER_NAME` | `nginx-proxy-manager` | App container name. |
| `DB_CONTAINER_NAME` | `nginx-proxy-manager-db` | Database container name. |
| `NPM_HTTP_PORT` | `80` | Host HTTP port. |
| `NPM_HTTPS_PORT` | `443` | Host HTTPS port. |
| `NPM_ADMIN_PORT` | `81` | Host admin UI port. |
| `DISABLE_IPV6` | `true` | Disables IPv6 inside NPM when the host does not provide it. |
| `MYSQL_DATABASE` | `npm` | MariaDB database name. |
| `MYSQL_USER` | `npm` | MariaDB application user. |
| `MYSQL_ROOT_PASSWORD` | - | Required MariaDB root password. |
| `MYSQL_PASSWORD` | - | Required MariaDB application password. |

Do not commit `.env`. Use `.env.example` only as a template.

## Persistent Storage

The Docker volume names are intentionally pinned. This prevents stack manager
project renames from creating fresh empty volumes.

| Volume | Mount | Contents |
|---|---|---|
| `nginx_data` | `/data` | NPM application data and SQLite migration metadata. |
| `nginx_letsencrypt` | `/etc/letsencrypt` | Certificates, renewal state, and ACME account data. |
| `nginx_mysql` | `/var/lib/mysql` | MariaDB data directory. |

After deployment, `docker volume ls` should show exactly:

```text
nginx_data
nginx_letsencrypt
nginx_mysql
```

It should not show dynamically generated names such as `<stack>_data`.

## Deploying with a Stack Manager

### Komodo

1. Create a new Stack.
2. Set the linked repository to `https://github.com/oehrn/Nginx`.
3. Use branch `main`.
4. Leave `file_paths` empty (Compose lives at repo root).
5. Open the Environment tab and paste the contents of `.env.example`.
6. Replace both database password placeholders.
7. Deploy.

### Portainer / Dockge

1. Add a stack from a Git repository.
2. Repository URL: `https://github.com/oehrn/Nginx`
3. Compose path: `docker-compose.yml`
4. Add environment variables or upload `.env`.
5. Deploy.

## Updating

Images are pinned deliberately. To update, change the image tags in
`docker-compose.yml`, review upstream release notes, and redeploy:

```bash
docker compose pull
docker compose up -d
```

## Backups

Back up all three volumes together while the stack is stopped or from a
consistent filesystem snapshot:

```bash
docker run --rm \
  -v nginx_data:/backup/nginx_data:ro \
  -v nginx_letsencrypt:/backup/nginx_letsencrypt:ro \
  -v nginx_mysql:/backup/nginx_mysql:ro \
  -v "$PWD":/out \
  alpine tar czf /out/nginx-proxy-manager-backup.tar.gz -C /backup .
```

Restore by stopping the stack, restoring the volume contents, and starting it
again.

## Useful Commands

```bash
docker compose ps
docker logs nginx-proxy-manager -f
docker logs nginx-proxy-manager-db -f
docker volume ls | grep '^local.*nginx_'
```

## License

MIT - see [LICENSE](LICENSE).
