# Mall Template

A lightweight, self-hostable online store template: a Spring Boot backend, a Vue 3 admin console and a Vue 3 storefront that run together with one Docker Compose command.

This repository is the entry point. It explains how the pieces fit together and holds the deployment example for the whole stack.

## Repositories

| Repository | What it is | Stack |
| --- | --- | --- |
| [mall-backend-template](https://github.com/dreaifeHebi/mall-backend-template) | Admin API (`mall-admin`) and storefront API (`mall-portal`) | Java 8, Spring Boot 2.7, PostgreSQL, Redis |
| [mall-admin-template](https://github.com/dreaifeHebi/mall-admin-template) | Admin console for catalog, orders, refunds, members and settings | Vue 3, Vite, TypeScript, Element Plus |
| [mall-user-template](https://github.com/dreaifeHebi/mall-user-template) | Storefront: browse, cart, checkout, payment, orders, refunds, account | Vue 3, Vite, TypeScript |
| **mall-template-docs** (this repo) | Overview, database notes and the Docker Compose deployment | Docker Compose, Nginx |

How they connect in the Docker deployment:

```text
                 ┌───────────────────────────┐
  :8060  ───────▶│ user-web   (Nginx)        │── /portal/, /uploads/ ──▶ mall-portal :8085 ──┐
                 │ mall-user-template build  │                                               │
                 └───────────────────────────┘                                               │
                 ┌───────────────────────────┐                                               ├──▶ PostgreSQL 16
  :8090  ───────▶│ admin-web  (Nginx)        │── /api/, /uploads/ ─────▶ mall-admin  :8080 ──┤    Redis 7
                 │ mall-admin-template build │                                               │
                 └───────────────────────────┘                                               │
  :8070  ───────▶  api-gateway (Nginx) ── /api/ → mall-admin, /portal/ + /uploads/ → mall-portal
```

Only the three Nginx entry points are published on the host. PostgreSQL, Redis and the two Java services stay on the internal Docker network.

## Features

- **Storefront**: home page, product list and detail, brands and categories, cart, checkout, payment, order history, refunds, account settings, addresses, favorites, brand follows and browsing history
- **Admin console**: products and SKUs, brands, categories, orders (export, edits, delivery, notes), refund review, members, admin users and roles, banners, email templates and system settings
- **Payments**: a built-in local provider (`LOCAL_SELF`) completes the checkout flow without any payment account; Stripe, Alipay, WeChat Pay and GlobePay have integration points in the backend
- **Read-only demo mode**: the seeded `demo_viewer` admin account can browse the catalog and settings but cannot see customer data or make changes
- **English first**: both frontends default to English and offer an `EN / 中文` switch
- **Small footprint**: only PostgreSQL and Redis are required; heavier upstream services (Elasticsearch, RabbitMQ, MongoDB, MinIO) are left out and can be added as extensions

## Tech Stack

- Backend: Java 8, Spring Boot 2.7.5, Spring Security + JWT, MyBatis, PageHelper
- Data: PostgreSQL 16, Redis 7, local file uploads
- Frontends: Vue 3, Vite 6, TypeScript, Pinia, Vue Router; Element Plus in the admin console
- Delivery: Docker Compose, Nginx 1.27, optional Cloudflare Tunnel, GitHub Actions for builds

## Quick Start

You need Docker with the Compose plugin. Nothing else has to be installed locally; the backend is built inside a Maven container.

### 1. Clone the four repositories side by side

```bash
mkdir mall-template && cd mall-template
git clone https://github.com/dreaifeHebi/mall-backend-template.git
git clone https://github.com/dreaifeHebi/mall-admin-template.git
git clone https://github.com/dreaifeHebi/mall-user-template.git
git clone https://github.com/dreaifeHebi/mall-template-docs.git
```

All following commands run from this `mall-template/` directory.

### 2. Build the backend jars

```bash
docker run --rm -v "$PWD/mall-backend-template:/workspace" -w /workspace maven:3.9.9-eclipse-temurin-21 \
  mvn -B -DskipTests clean package -pl mall-admin,mall-portal -am

cp mall-backend-template/mall-admin/target/mall-admin-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-admin.jar
cp mall-backend-template/mall-portal/target/mall-portal-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-portal.jar
```

### 3. Create the environment file

```bash
cp mall-template-docs/deploy/local-docker/.env.example mall-template-docs/deploy/local-docker/.env
```

Change `POSTGRES_PASSWORD` and `REDIS_PASSWORD` in `.env` before using the stack anywhere other than your own machine.

### 4. Start the stack

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```

The first start initializes the database with a small English sample catalog. Then open:

| Entry point | URL |
| --- | --- |
| Storefront | `http://localhost:8060` |
| Admin console | `http://localhost:8090` |
| API gateway | `http://localhost:8070` (health check: `/health`) |

### Stop or reset

```bash
# stop
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down

# stop and delete all data (the database is re-initialized on the next start)
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down -v
```

More detail, including the optional Cloudflare Tunnel, is in [deploy/local-docker/README.md](./deploy/local-docker/README.md).

## Demo Accounts

The database initializer creates these accounts:

| Account | Password | Where | Notes |
| --- | --- | --- | --- |
| `demo_user` | `123456` | Storefront | Regular customer account |
| `demo_viewer` | `123456` | Admin console | Read-only: catalog, banners, settings and email templates |
| `admin` | `123456` | Admin console | Super admin; change this password before going public |

Both login pages offer a one-click fill for the demo account.

## Configuration

All settings for the Compose deployment live in [`deploy/local-docker/.env.example`](./deploy/local-docker/.env.example):

| Variable | Purpose |
| --- | --- |
| `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` | Database name and credentials (passwords are required) |
| `REDIS_PASSWORD` | Redis password (required) |
| `USER_WEB_PORT`, `ADMIN_WEB_PORT`, `API_WEB_PORT` | Host ports for storefront, admin console and API gateway (8060 / 8090 / 8070) |
| `MALL_DOMAIN`, `FRONTEND_BASE_URL` | Public URLs used in generated links and emails; set them to your real domain when deploying |
| `MALL_UPLOAD_PATH`, `MALL_UPLOAD_URL` | Upload directory (a shared Docker volume) and its URL prefix |
| `SPRING_MAIL_FROM`, `SES_SMTP_USERNAME`, `SES_SMTP_PASSWORD` | Outgoing email, such as registration confirmation and password reset |
| `GLOBEPAY_*` | Optional GlobePay payment settings |
| `JAVA_OPTS` | JVM memory settings for the two Java services |

Backend and frontend settings for running each part on its own are described in the individual repositories.

## Deployment Options

| Option | When to use | Where |
| --- | --- | --- |
| All-in-one Docker Compose | Local evaluation, LAN demo, a single small server | [`deploy/local-docker/`](./deploy/local-docker/) |
| Compose + Cloudflare Tunnel | Expose the single-host stack on public hostnames without opening ports | [`docker-compose.cloudflare.yml`](./deploy/local-docker/docker-compose.cloudflare.yml) and [`cloudflared-config.example.yml`](./deploy/local-docker/cloudflared-config.example.yml) |
| Backend only | Run the APIs with PostgreSQL and Redis, frontends elsewhere | [`deployment/`](https://github.com/dreaifeHebi/mall-backend-template/tree/main/deployment) in mall-backend-template |
| Static frontends on S3-compatible hosting | Serve the two frontends from object storage and point them at the API gateway | Manual GitHub Actions workflows in mall-admin-template and mall-user-template |

The frontend workflows only build by default. To deploy, run them with `deploy` set to `true` after configuring the secret `AWS_ROLE_ARN` and these repository variables:

- `AWS_REGION`
- `ADMIN_S3_BUCKET`, `ADMIN_BASE_API`
- `USER_S3_BUCKET`, `USER_API_BASE_URL`, `USER_FRONTEND_BASE_URL`, `USER_BACKEND_BASE_URL`

The backend repository's CI builds both jars and uploads them as a workflow artifact.

## Known Limitations

- The default `LOCAL_SELF` payment provider marks orders as paid immediately. Real payment providers need merchant credentials and callback testing.
- Refunds are reviewed in the admin console; real refunds to the original payment method depend on a configured payment provider.
- Emails are only sent when SMTP credentials are configured.

## Documentation

- [DATABASE.md](./DATABASE.md): schema overview, seed data and how to re-initialize the database
- [deploy/local-docker/README.md](./deploy/local-docker/README.md): deployment details and the optional Cloudflare Tunnel
- [docs/migration-notes.md](./docs/migration-notes.md): how the template set was extracted from upstream mall (for maintainers)

## License

This repository does not include a LICENSE file yet. For the code, see each repository: mall-backend-template and mall-admin-template are licensed under Apache-2.0.

The template set is derived from [macrozheng/mall](https://github.com/macrozheng/mall) (Apache-2.0) and its frontends [mall-admin-web](https://github.com/macrozheng/mall-admin-web) and [mall-app-web](https://github.com/macrozheng/mall-app-web). Thanks to the original authors.
