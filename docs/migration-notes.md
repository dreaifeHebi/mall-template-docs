# Migration Notes

Background on how this template set was extracted from the original mall projects, kept for maintainers. For the public overview, see the [README](../README.md).

Ongoing migration progress, open items and verification logs belong in this file (or in each repository's own `docs/migration-notes.md`), not in the READMEs.

Other maintainer-only files in this repository:

- [`PROJECT_CONTEXT.md`](../PROJECT_CONTEXT.md): detailed working context and verification history.
- [`PUBLISHING.md`](../PUBLISHING.md): checklist for stripping local configs, artifacts and machine-specific values before publishing.

## Extraction

This template set is extracted from the original mall backend, admin frontend, and user frontend repositories.

- `mall-backend-template`: Spring Boot mall backend core.
- `mall-admin-template`: Vue 3 + Vite + TypeScript admin template. The old Vue 2 + Webpack code is archived under `legacy-vue2/` and is not part of the runtime build.
- `mall-user-template`: Vue 3 + Vite + TypeScript storefront template. The old UniApp code is archived under `legacy-uniapp/`; runtime hash compatibility has been removed.

## PostgreSQL Switch

The backend now defaults to PostgreSQL:

- Maven driver: `org.postgresql:postgresql`
- JDBC URL: `jdbc:postgresql://...`
- PageHelper dialect: `postgresql`
- Deployment database service: `postgres:16-alpine`
- Initialization SQL: `deployment/postgres/init/01-init-database.sql`

## Next Modernization Step

The following is the "Next Modernization Step" section from the previous README, kept as-is.

The Vue 3 shells already exist. Keep optional integrations outside the lightweight core:

- Admin: product list/create/edit/status flows, brand/category management, mall member management, administrator management, marketing banners, system settings, role permissions, email templates, order list/detail, receiver/money edits, close/delete, confirm payment, delivery, notes, refund review/process pages, read-only public demo mode, default English UI, and `EN / 中文` language switching have been migrated into the Vue 3 admin.
- Storefront: Stripe/Alipay/WeChat production provider configuration, real payment callbacks, and real original-route refund integration remain production integration items. Address management, favorites, session-backed browsing history, brand attention, brand list/detail, new/hot product list entries, cart checkout, order list, order detail, account profile/password/email verification, content/help pages, the local H5 payment record flow, `LOCAL_SELF` self-confirmed payment success, member refund/after-sales pages, default English UI, and `EN / 中文` language switching have been migrated into the Vue 3 storefront.

## Publishing

Before publishing changes from a local workspace, use [`PUBLISHING.md`](../PUBLISHING.md) to strip local configs, generated artifacts, and machine-specific values.
