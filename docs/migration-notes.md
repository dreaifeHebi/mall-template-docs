# Migration Notes

Background on how Mall Template was extracted from the original mall projects, kept for maintainers. For the public overview, see the [README](../README.md).

Ongoing migration progress, open items and verification logs belong in this file (or in each repository's own `docs/migration-notes.md`), not in the READMEs.

## Extraction

Mall Template was extracted from the backend, admin frontend and user frontend repositories of a production store built on [macrozheng/mall](https://github.com/macrozheng/mall).

- `mall-backend-template`: Spring Boot mall backend core.
- `mall-admin-template`: Vue 3 + Vite + TypeScript admin template. The old Vue 2 + Webpack code was removed from the repository after the migration.
- `mall-user-template`: Vue 3 + Vite + TypeScript storefront template. The old UniApp code was removed from the repository after the migration, and runtime hash compatibility has been removed.

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
