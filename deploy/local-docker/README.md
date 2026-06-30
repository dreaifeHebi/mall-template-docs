# Local Docker Deployment

This example deploys the four public template repositories as sibling checkouts.

Expected directory layout:

```text
workspace/
  mall-admin-template/
  mall-backend-template/
  mall-template-docs/
  mall-user-template/
```

## Build Backend Jars

Run from `workspace/`:

```bash
docker run --rm -v "$PWD/mall-backend-template:/workspace" -w /workspace maven:3.9.9-eclipse-temurin-21 \
  mvn -B -DskipTests clean package -pl mall-admin,mall-portal -am

cp mall-backend-template/mall-admin/target/mall-admin-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-admin.jar
cp mall-backend-template/mall-portal/target/mall-portal-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-portal.jar
```

The copied jars are runtime artifacts and should not be committed.

## Configure

```bash
cp mall-template-docs/deploy/local-docker/.env.example mall-template-docs/deploy/local-docker/.env
```

Edit `.env` before sharing or deploying outside a local machine.

## Start

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```

Entry points:

- Storefront: `http://localhost:8060`
- Admin: `http://localhost:8090`
- API gateway: `http://localhost:8070`

## Stop

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down
```

To reset local data:

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down -v
```

## Optional Cloudflare Tunnel

Cloudflare Tunnel is intentionally split into an optional Compose override. The
base deployment does not require Cloudflare.

The tunnel config is example-only. Do not commit real tunnel IDs, credential
paths, or credential JSON files.

```bash
cp mall-template-docs/deploy/local-docker/cloudflared-config.example.yml \
  mall-template-docs/deploy/local-docker/cloudflared-config.yml
```

Edit hostnames, tunnel id, and credentials path. If credentials live somewhere
other than `./credentials`, set `CLOUDFLARED_CREDENTIALS_DIR` in your shell or
local `.env`.

Then run:

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml \
  -f mall-template-docs/deploy/local-docker/docker-compose.cloudflare.yml \
  up -d api-gateway cloudflared
```
