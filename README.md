# Mall Template Extraction

This template set is extracted from the original mall backend, admin frontend, and user frontend repositories.

## Repositories

- `mall-backend-template`: Spring Boot mall backend core.
- `mall-admin-template`: Vue 3 + Vite + TypeScript admin template. The old Vue 2 + Webpack code is archived under `legacy-vue2/` and is not part of the runtime build.
- `mall-user-template`: Vue 3 + Vite + TypeScript storefront template. The old UniApp code is archived under `legacy-uniapp/`; runtime hash compatibility has been removed.

## Backend Core

The backend core keeps only the services required for a lightweight mall starter:

- PostgreSQL
- Redis
- `mall-admin`
- `mall-portal`
- local file upload

The default deployment intentionally excludes:

- MinIO container
- RabbitMQ
- Elasticsearch / `mall-search`
- MongoDB
- Logstash / ELK
- historical deployment scripts tied to a specific server

## Optional Extensions

Put optional integrations under `mall-backend-template/extensions/` when they are needed:

- `storage-s3`: S3, Cloudflare R2, or external MinIO.
- `storage-aliyun-oss`: Aliyun OSS policy upload.
- `search-elasticsearch`: advanced product search.
- `messaging-rabbitmq`: queue-backed delayed order cancellation.

The core template should stay runnable without those extensions.

## PostgreSQL

The backend now defaults to PostgreSQL:

- Maven driver: `org.postgresql:postgresql`
- JDBC URL: `jdbc:postgresql://...`
- PageHelper dialect: `postgresql`
- Deployment database service: `postgres:16-alpine`
- Initialization SQL: `deployment/postgres/init/01-init-database.sql`

## Deployment

The public Docker deployment example lives in:

```text
mall-template-docs/deploy/local-docker/
```

It expects the four repositories to be checked out as siblings:

```text
workspace/
  mall-admin-template/
  mall-backend-template/
  mall-template-docs/
  mall-user-template/
```

Run from `workspace/`:

```bash
docker run --rm -v "$PWD/mall-backend-template:/workspace" -w /workspace maven:3.9.9-eclipse-temurin-21 \
  mvn -B -DskipTests clean package -pl mall-admin,mall-portal -am

cp mall-backend-template/mall-admin/target/mall-admin-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-admin.jar
cp mall-backend-template/mall-portal/target/mall-portal-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-portal.jar

cp mall-template-docs/deploy/local-docker/.env.example mall-template-docs/deploy/local-docker/.env

docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```

The stack exposes these entrypoints to the host/LAN:

- Storefront: `http://localhost:8060`
- Admin: `http://localhost:8090`
- API gateway: `http://localhost:8070`
- Admin API: internal service `mall-admin:8080`, proxied by admin Nginx under `/api/`
- Portal API: internal service `mall-portal:8085`, proxied by storefront Nginx under `/portal/`
- PostgreSQL: internal service `postgres:5432`
- Redis: internal service `redis:6379`

Default accounts:

- Storefront: `demo_user / 123456`
- Admin: `admin / 123456`
- Admin read-only public demo: `demo_viewer / 123456`

Frontend cloud workflows are manual by default. Set repository variables and secrets before enabling deploy:

- `AWS_ROLE_ARN`
- `AWS_REGION`
- `ADMIN_S3_BUCKET`
- `USER_S3_BUCKET`
- `ADMIN_BASE_API`
- `USER_API_BASE_URL`
- `USER_FRONTEND_BASE_URL`
- `USER_BACKEND_BASE_URL`

Before publishing these repositories publicly, use
[`PUBLISHING.md`](./PUBLISHING.md) to strip local configs, generated artifacts,
and machine-specific values.

## Next Modernization Step

The Vue 3 shells already exist. Keep optional integrations outside the lightweight core:

- Admin: product list/create/edit/status flows, brand/category management, mall member management, administrator management, marketing banners, system settings, role permissions, email templates, order list/detail, receiver/money edits, close/delete, confirm payment, delivery, notes, refund review/process pages, read-only public demo mode, default English UI, and `EN / 中文` language switching have been migrated into the Vue 3 admin.
- Storefront: Stripe/Alipay/WeChat production provider configuration, real payment callbacks, and real original-route refund integration remain production integration items. Address management, favorites, session-backed browsing history, brand attention, brand list/detail, new/hot product list entries, cart checkout, order list, order detail, account profile/password/email verification, content/help pages, the local H5 payment record flow, `LOCAL_SELF` self-confirmed payment success, member refund/after-sales pages, default English UI, and `EN / 中文` language switching have been migrated into the Vue 3 storefront.
