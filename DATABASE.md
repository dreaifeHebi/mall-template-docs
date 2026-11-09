# Database Shape

The lightweight template uses PostgreSQL as the default database. The initializer lives at:

```text
mall-backend-template/deployment/postgres/init/01-init-database.sql
```

## Default Accounts

- Storefront demo: `demo_user / 123456`
- Admin: `admin / 123456`
- Admin public read-only demo: `demo_viewer / 123456`

`demo_viewer` is seeded with a precise read-only resource set for catalog, banners, settings, email templates, dashboard, login info, and logout. It is not granted order, refund, member, admin-user, role, menu, or resource-management permissions.

## Active Business Areas

- `ums_admin`, `ums_role`, `ums_resource`, relation tables: admin login, role/resource authorization, and read-only demo account.
- `ums_member`, `ums_member_receive_address`: storefront account, profile, email status, and shipping addresses.
- `pms_brand`, `pms_product_category`, `pms_product`, `pms_sku_stock`, product attribute tables: catalog, brand/category navigation, product detail, and stock.
- `oms_cart_item`, `oms_order`, `oms_order_item`, `oms_order_operate_history`, `oms_order_return_apply`: cart, checkout, order lifecycle, and refund/after-sales flow.
- `oms_payment_method`, `oms_payment_record`, `oms_payment/refund` related tables: local H5 payment record flow. `LOCAL_SELF` is the default template provider.
- `sms_home_advertise`, home/recommend/subject tables, `cms_banner`: storefront home content and banners.
- `sms_email_template`, `sys_setting`: email templates and editable runtime settings.
- `ums_member_product_collection`, `ums_member_read_history`, `ums_member_brand_attention`: logged-in member interactions. Anonymous read history is handled in browser `sessionStorage`, not the database.

## Default Seed Data

The seed data is intentionally small and English-first:

- Brands: Nova Supply, Urban Nest, Luma Works.
- Categories: Electronics, Home Goods, Everyday Carry.
- Products: Nova Keys Wireless Keyboard, Urban Sound Mini Speaker, Luma Commuter Backpack.
- Images: local `/template-assets/*.svg` files, avoiding external placeholder hosts.
- Email templates: order confirmation, shipment update, registration confirmation, order edit reminder, and password reset.

## Disabled By Default

Member levels and points are no longer part of the default template surface:

- No member-level seed rows are inserted.
- Product/admin/order UI does not expose level or points controls.
- Historical compatibility tables/fields such as `ums_member_level`, `ums_integration_change_history`, and `ums_integration_consume_setting` remain so the current backend model can run without a wider generated-model rewrite.

## Reinitialization

To recreate the database from the initializer:

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down -v

docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```
