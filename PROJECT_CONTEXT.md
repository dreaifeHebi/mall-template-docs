# Mall Template Context

本文档记录当前 mall template 的结构、技术栈、部署方式和本地验证结果，供后续 Codex/维护会话快速接续。

## 文档约定

- 各仓库的 `README.md` 只写面向外部访客的内容：介绍、在模板中的位置、功能、技术栈、快速开始、配置、已知限制、许可证。
- 迁移进度、待办和验证记录不要写进 README：跨仓库的写在本文件或 `docs/migration-notes.md`，单个前端仓库的写在该仓库的 `docs/migration-notes.md`。
- README 不引导外部访客去看 `PUBLISHING.md`、本文件这类维护者文档。

## 目录结构

当前统一管理目录：

```text
<workspace>/
  mall-backend-template/    # Spring Boot 后端模板
  mall-admin-template/      # Vue 3 + Vite 管理端模板
  mall-user-template/       # Vue 3 + Vite 商城用户端模板
  mall-template-docs/       # 上下文、方案文档和公开部署示例
    deploy/local-docker/    # 四个 repo 同级 checkout 时使用的 Docker Compose 示例
```

当前本机工作区还保留 `local-deploy/` 作为运行环境入口，但它不作为公开拆分 repo 的发布内容。

原始 clone 备份目录曾保留在本机模板工作区之外，未移动到公开模板目录：

```text
mall-admin-web-master/
mall-app-web-front/
smoke-mall/
```

## 技术栈

后端：

- Java 21 用于本地构建，运行镜像当前使用 `eclipse-temurin:8-jre-alpine`
- Spring Boot 2.7.x
- PostgreSQL 16
- Redis 7
- MyBatis / PageHelper
- 本地文件上传路径：`/data/uploads`

管理端：

- Vue 3
- Vite
- TypeScript
- Element Plus
- Pinia
- Nginx 静态部署，`/api/` 反代到 `mall-admin:8080`

用户端：

- Vue 3
- Vite
- TypeScript
- Pinia
- Nginx 静态部署，`/portal/` 反代到 `mall-portal:8085`

## 本地 Docker 部署

部署入口：

```bash
cd <workspace>
```

先构建后端 jar：

```bash
docker run --rm -v "$PWD/mall-backend-template:/workspace" -w /workspace maven:3.9.9-eclipse-temurin-21 \
  mvn -B -DskipTests clean package -pl mall-admin,mall-portal -am

cp mall-backend-template/mall-admin/target/mall-admin-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-admin.jar
cp mall-backend-template/mall-portal/target/mall-portal-1.0-SNAPSHOT.jar mall-backend-template/deployment/mall-portal.jar
```

启动：

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```

停止：

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down
```

重置数据库并重新启动：

```bash
docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml down -v

docker compose --env-file mall-template-docs/deploy/local-docker/.env \
  -f mall-template-docs/deploy/local-docker/docker-compose.yml up -d --build
```

## 访问地址

本机：

- 用户端：`http://localhost:8060/`
- 管理端：`http://localhost:8090/`

局域网：

- 用户端：`http://<host-lan-ip>:8060/`
- 管理端：`http://<host-lan-ip>:8090/`

当前验证机器的 LAN IP 是：

```text
<host-lan-ip>
```

所以当前可用地址：

- 用户端：`http://<host-lan-ip>:8060/`
- 管理端：`http://<host-lan-ip>:8090/`

默认管理端账号：

```text
admin / 123456
```

默认前台展示账号：

```text
demo_user / 123456
```

默认管理端只读展示账号：

```text
demo_viewer / 123456
```

`demo_viewer` 只用于对外展示，前端会隐藏订单、退款、会员、管理员、角色权限等包含用户或敏感信息的页面，并拦截非 GET 写操作。

## Compose 服务

`mall-template-docs/deploy/local-docker/docker-compose.yml` 启动以下服务：

- `postgres`: PostgreSQL 16，仅 Docker 内部网络
- `redis`: Redis 7，仅 Docker 内部网络
- `mall-admin`: 管理端 API，容器内 `8080`
- `mall-portal`: 用户端 API，容器内 `8085`
- `admin-web`: 管理端 Nginx，对外 `0.0.0.0:8090`
- `user-web`: 用户端 Nginx，对外 `0.0.0.0:8060`

PostgreSQL 和 Redis 不暴露到 LAN；LAN 只访问两个 Nginx 入口。

## 已做的模板化调整

- 移除重型外部依赖方向：不再要求 MinIO、Elasticsearch、MongoDB、RabbitMQ 参与本地模板部署。
- 数据库切换到 PostgreSQL。
- 管理端从 Vue 2/Webpack 替换为 Vue 3/Vite/TypeScript。
- 用户端从 UniApp 替换为普通 Vue 3/Vite/TypeScript。
- 前台和管理端均已接入轻量 locale 层：默认语言为英文，顶部提供 `EN / 中文` 切换，用户选择会写入 `localStorage`；中文模式保留原中文源码文案，英文模式会翻译页面 UI 文案、按钮、表头、placeholder、title、aria-label、原生确认弹窗和动态插入的 Element Plus 文案。数据库里的商品名、品牌名等业务数据不在运行时强行翻译。
- 用户端已继续迁移收货地址管理：列表、添加、编辑、删除、设为默认和英国手机号/区号字段。
- 用户端已继续迁移会员互动：商品收藏、浏览足迹、品牌关注；对应 PostgreSQL 表为 `ums_member_product_collection`、`ums_member_read_history`、`ums_member_brand_attention`。
- 用户端已继续迁移品牌详情：`/brands/{id}` 展示品牌大图、Logo、首字母、品牌故事、关注品牌和相关商品分页；首页推荐品牌和关注品牌列表已链接到品牌详情。
- 用户端已继续迁移品牌/新热商品入口：`/brands` 展示推荐品牌列表，`/products/new` 和 `/products/hot` 复用商品列表页并按新品/销量切换默认排序；首页品牌和商品区块已链接到对应入口。
- 用户端已继续迁移基础内容页：`/about`、`/contact`、`/help`、`/notice` 提供关于商城、客服联系方式、帮助中心和下单须知；运行层不再做旧 UniApp hash 路径兼容。
- 用户端已对照旧 UniApp `pages.json` 做最终页面缺口扫描；正式旧页面均有新 Vue 3 页面或明确的轻量模板替代入口，旧 UniApp 代码仅保留为离线参考。
- 用户端已继续迁移订单中心：订单列表状态筛选、订单详情、取消订单、确认收货和删除订单。
- 用户端已继续迁移确认下单链路：购物车“提交订单”进入 `/checkout`，确认页会把本地选中购物车同步到后端购物车，加载收货地址、促销金额和商品清单，并通过 `/portal/order/generateOrder` 创建待付款订单后跳转 H5 支付页。
- 用户端已继续迁移本地 H5 支付记录链路：`/orders/{id}/pay` 支持支付方式选择、创建支付记录、查询支付状态和取消支付；当前模板默认使用 `LOCAL_SELF` 本地自确认 provider，创建支付记录后直接调用本地订单支付成功流程，不需要第三方跳转；`STRIPE_CHECKOUT`、`ALIPAY_H5`、`WECHAT_H5`、`CREDIT_CARD` 保留为支付方式/生产 provider 扩展位，真实外部支付和回调需配置对应 provider、密钥和回调。
- 用户端已继续迁移退款售后链路：`/orders/{id}/refund`、`/refunds`、`/refunds/{id}` 支持退款申请、列表筛选、详情/状态刷新和待审核取消；真实原路退款仍需后台审核和支付环境配置。
- 用户端已继续迁移账户设置/密码/邮箱验证链路：`/account/settings` 支持资料编辑、发送邮箱验证和修改密码；`/auth/forgot-password`、`/auth/reset-password`、`/auth/email-confirm` 支持找回密码、重置密码和邮箱确认。
- 管理端已继续迁移订单管理：`/orders`、`/orders/{id}` 支持订单状态/日期/收货人筛选、详情、当前筛选导出、修改收货信息、修改费用信息、关闭订单、删除订单、后台确认收款、发货、备注、商品明细、收货/支付/物流信息和操作记录。
- 管理端已继续迁移退款审核：`/refunds`、`/refunds/{id}` 支持退款单/订单号/会员/状态/日期筛选、详情、审核通过/拒绝、处理退款和状态刷新。
- 管理端已继续迁移商品管理：`/catalog/products`、`/catalog/products/new`、`/catalog/products/{id}/edit` 支持商品筛选、导出、新增、编辑、上下架、新品、推荐、软删除、SKU 库存和高级关联参数 JSON 透传。
- 管理端已继续迁移品牌/分类管理：`/catalog/brands` 支持品牌筛选、新增、编辑、Logo/大图上传、品牌副标题/故事、显示/制造商状态和批量操作；`/catalog/categories` 支持一级/二级分类切换、新增、编辑、图标上传、导航/显示状态、批量状态和筛选属性关联。
- 管理端已继续迁移用户/管理员管理：`/users/members` 支持商城会员筛选、资料编辑、邮箱验证、状态冻结、批量冻结和删除；`/system/admins` 支持后台管理员筛选、新增、编辑、启停、删除和角色分配/清空；公开展示只读账号隐藏用户/管理员敏感页面。
- 前台商品详情已增加品牌与分类跳转；未登录浏览历史改为本次浏览器 session 存储，登录用户仍使用后端浏览足迹。
- 前台和管理端已清理会员等级/积分展示；底层兼容表和历史字段保留但默认初始化不写入会员等级/积分种子。
- 管理端已继续迁移营销/系统设置：`/marketing/banners` 支持轮播图列表、新增、编辑、图片上传/URL、跳转链接、排序、启停和删除；`/system/settings` 支持设置筛选、新增、编辑、取值、启停、删除、批量删除、获取全部启用设置和批量更新。
- 管理端已继续迁移角色权限：`/system/roles` 支持角色筛选、新增、编辑、启停、删除、菜单分配和资源分配。
- 管理端已继续迁移邮件模板：`/system/email-templates` 支持模板筛选、新增、编辑、启停、单个删除和批量删除。
- `mall-portal` 的生产 profile 安全白名单已从 `/**` 收紧为公开接口白名单，会员中心类接口恢复 JWT 保护，未登录访问返回业务 `code: 401`。
- `mall-portal` 订单相关接口已补充会员归属校验，避免登录会员读取或操作他人订单；购物车促销计算补齐后端可信价格兜底，避免订单生成时 `CartPromotionItem.price` 为空；订单生成对空购物车和不存在的收货地址返回业务失败，不再抛空指针。
- `mall-portal` H5 支付创建已校验订单归属、待支付状态、订单号和金额，支付方式映射补齐 `createTime/updateTime` 字段，避免 `/payment/h5/methods` 读取失败。
- `mall-portal` 支付 provider 默认值为 `LOCAL_SELF`；本地支付会写入支付记录、生成本地第三方单号并将记录标记为 `SUCCESS`，同时保留 `STRIPE_CHECKOUT`、支付宝 H5、微信 H5 和信用卡渠道的生产扩展入口。
- `mall-portal` 支付成功流程已补无 SKU 商品库存兜底：Vue 前台购物车允许直接用商品 ID 加购时，支付成功会按 `productId` 扣减商品库存；有 SKU 时仍按 `productSkuId` 扣减 SKU 和商品库存。该流程加了事务边界，避免库存扣减失败时订单状态半成功。
- `mall-portal` 已放行内部后台确认支付接口 `/payment/h5/admin/confirm`，供 `mall-admin` 在本地模板中确认待付款订单。
- `mall-portal` 退款接口已改为从当前 JWT 会员读取身份，不再信任 `Member-Id` 请求头；退款申请只允许待发货、已发货、已完成订单，避免已关闭/无效订单进入退款申请。
- `mall-portal` 注册确认邮件、密码重置邮件和邮箱验证邮件已改为指向 Vue 3 前台路由；注册确认 token 会写入 Redis；修改密码后会清除会员缓存，避免接口返回成功但新密码无法登录。
- `mall-admin` 退款分页改为返回真实 `CommonPage`，补齐退款 DAO 中会员列表、统计和批量查询映射；退款成功后的全额判断改为按实付金额 `payAmount`，避免含运费订单被商品金额退款误关闭。
- `mall-admin` 订单费用修改会重新计算 `payAmount`；后台关闭待付款订单时按前台取消订单语义释放 SKU 锁定库存。
- 新增 Dockerfile 和 Nginx 配置。
- Docker Compose 示例的默认 `JAVA_OPTS` 已修正为无嵌套引号，避免重建 Java 容器后把 `"-Xmx320m` 当成主类启动失败。
- Nginx 使用 Docker 内置 DNS 动态解析后端服务，后端容器重建后不会因缓存旧 IP 导致 502。
- PostgreSQL 初始化脚本补齐首页广告、首页推荐、专题、秒杀等模板运行所需的最小表结构和样例数据。
- 修复当前运行路径中残留的 MySQL 方言 SQL，包括反引号、`date_add`、`DATE_SUB`、`CURDATE`、`DATE_FORMAT`、`LIMIT offset, limit` 和 `UPDATE JOIN`。

## 验证记录

本地 Docker 栈已验证：

- `docker compose ps`: 6 个服务均 `Up`，PostgreSQL/Redis 为 healthy。
- `http://<host-lan-ip>:8060/`: HTTP 200。
- `http://<host-lan-ip>:8090/`: HTTP 200。
- `http://<host-lan-ip>:8060/portal/home/content`: HTTP 200，业务 `code: 200`。
- `POST http://<host-lan-ip>:8090/api/admin/login`，账号 `admin / 123456`: HTTP 200，业务 `code: 200`。
- 2026-06-29 继续迁移验证：`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过；`mall-portal` 相关模块执行 `mvn -DskipTests compile -pl mall-portal -am` 通过。
- 2026-06-29 会员互动迁移验证：`mall-user-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-portal` 执行 `mvn -DskipTests package -pl mall-portal -am` 通过；本地 Docker 已重建 `mall-portal` 和 `user-web`。
- 2026-06-29 品牌关注接口验证：未登录访问 `/portal/member/attention/list` 返回业务 `code: 401`；本地测试会员可登录后完成 `/member/attention/add`、`detail`、`list`、`delete`，删除后列表为空。
- 2026-06-29 订单中心迁移验证：`mall-user-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-portal` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-portal -am` 通过，并已重建 `mall-portal` 容器。测试会员完成注册、登录、收货地址、购物车、`/portal/order/generateOrder`、`/portal/order/list`、`/portal/order/detail/{orderId}`、`/portal/order/cancelUserOrder`、`/portal/order/deleteOrder` 链路；未登录订单列表返回业务 `code: 401`；`/orders` 和 `/orders/{orderId}` 前端路由返回 HTTP 200；取消后订单状态为 `4`，删除后列表为空；测试数据已清理，`pms_sku_stock.id=1` 的 `lock_stock` 回到 `0`。
- 2026-06-29 品牌详情迁移验证：`mall-user-template` 执行 `npm run typecheck`、`npm run build` 通过；本地 Docker 已重建 `user-web`。`/portal/brand/detail/1` 返回业务 `code: 200`；`/portal/brand/productList?brandId=1&pageNum=1&pageSize=8` 返回业务 `code: 200` 且 `total=3`；`/` 和 `/brands/1` 前端路由返回 HTTP 200。
- 2026-06-29 H5 支付迁移验证：`mall-user-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-portal` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-portal -am` 通过，并已重建 `mall-portal` 容器。未登录访问 `/portal/payment/h5/methods` 返回业务 `code: 401`；测试会员完成注册、登录、收货地址、购物车、`/portal/order/generateOrder`、`/portal/payment/h5/methods`、金额不匹配拦截、`/portal/payment/h5/create`、`/portal/payment/h5/status/{orderSn}/WECHAT_H5`、`/portal/payment/h5/cancel` 链路；支付记录状态从 `PENDING` 变为 `CANCELLED`；`/orders/{orderId}/pay` 前端路由返回 HTTP 200；测试数据已清理，`ums_member`/`oms_order`/`oms_payment_record` 无残留，`pms_sku_stock.id=1` 的 `lock_stock` 回到 `0`。
- 2026-06-29 退款售后迁移验证：`mall-user-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-portal` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-portal -am` 通过，并已重建 `mall-portal` 和 `user-web`。未登录访问 `/portal/refund/list` 返回业务 `code: 401`；测试会员完成注册、登录、收货地址、购物车、`/portal/order/generateOrder`、`/portal/order/paySuccess`、退款金额超限拦截、`/portal/refund/apply`、`/portal/refund/list`、`/portal/refund/{refundRequestId}`、`/portal/refund/status/{refundRequestId}`、`/portal/refund/cancel/{refundRequestId}` 链路；退款状态从 `PENDING_AUDIT` 变为 `CANCELLED`；`/orders/{orderId}/refund`、`/refunds`、`/refunds/{refundRequestId}` 前端路由返回 HTTP 200；测试数据已清理，`ums_member`/`oms_order`/`refund_request` 无残留，`pms_sku_stock.id=1` 恢复到 `stock=100, lock_stock=0`。
- 2026-06-29 管理端订单/退款迁移验证：`mall-admin-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-admin` 和 `mall-portal` 在 `/tmp` 干净构建副本分别执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-admin -am`、`mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-portal -am` 通过，并已重建 `mall-admin`、`mall-portal`、`admin-web`。admin 登录后 `/api/order/list`、`/api/order/{id}`、`/api/admin/refund/list`、`/api/admin/refund/{id}` 返回业务 `code: 200`；测试订单退款从 `PENDING_AUDIT` 经后台审核到 `SUCCESS`，部分退款后订单保持待发货并可通过 `/api/order/delivery/{id}` 发货，最终订单状态为 `2`；待付款订单通过 `/api/order/confirm/pay` 从状态 `0` 变为 `1`；`/orders`、`/orders/{id}`、`/refunds`、`/refunds/{id}` 前端路由返回 HTTP 200；测试数据已清理，`ums_member`/`oms_order`/`refund_request` 无残留，`pms_sku_stock.id=1` 恢复到 `stock=100, lock_stock=0`。
- 2026-06-29 管理端订单高级操作迁移验证：`mall-admin-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-admin` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-admin -am` 通过，并已重建 `mall-admin`、`admin-web`。测试会员经前台注册、收货地址、购物车和下单生成待付款订单后，后台 `/api/order/update/receiverInfo`、`/api/order/update/moneyInfo`、`/api/order/update/close`、`/api/order/delete` 均返回业务 `code: 200`；费用修改后应付金额按 `totalAmount + freightAmount - promotionAmount - integrationAmount - couponAmount - discountAmount` 重算为 `70.00`；关闭后订单状态为 `4`，删除后不再出现在订单列表；`/orders`、`/orders/{id}` 前端路由返回 HTTP 200；测试数据已清理，`ums_member`/`oms_order`/`refund_request` 无残留，`pms_sku_stock.id=1` 恢复到 `stock=100, lock_stock=0`。
- 2026-06-29 管理端商品管理迁移验证：`mall-admin-template` 执行 `npm run typecheck`、`npm run build` 通过，并已重建 `admin-web`。admin 登录后读取 `/api/brand/list`、`/api/productCategory/list/withChildren`，通过 `/api/product/create` 创建临时商品，再通过 `/api/product/updateInfo/{id}`、`/api/product/update/{id}`、`/api/product/update/publishStatus`、`/api/product/update/recommendStatus`、`/api/product/update/newStatus`、`/api/product/update/deleteStatus` 完成编辑、状态切换和软删除；软删除后商品不再出现在 `/api/product/list`；`/catalog/products`、`/catalog/products/new`、`/catalog/products/1/edit` 前端路由返回 HTTP 200；测试商品及关联 SKU/关系数据已清理。
- 2026-06-29 管理端角色权限/邮件模板迁移验证：`mall-admin-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-admin` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-admin -am` 通过，并已重建 `mall-admin`、`admin-web`。admin 登录后通过 `/api/role/create`、`/api/role/update/{id}`、`/api/role/updateStatus/{id}`、`/api/role/allocMenu`、`/api/role/allocResource`、`/api/role/listMenu/{id}`、`/api/role/listResource/{id}`、`/api/role/delete` 完成角色创建、编辑、启停、菜单/资源分配和删除；通过 `/api/smsEmailTemplate/create`、`/api/smsEmailTemplate/update/{id}`、`/api/smsEmailTemplate/updateStatus`、`/api/smsEmailTemplate/{id}`、`/api/smsEmailTemplate/delete/batch` 完成邮件模板创建、编辑、启停、详情和批量删除；`/system/roles`、`/system/email-templates` 前端路由返回 HTTP 200；测试角色和测试邮件模板已清理。
- 2026-06-29 管理端品牌/分类迁移验证：`mall-admin-template` 执行 `npm run typecheck`、`npm run build` 通过；`mall-admin` 在 `/tmp` 干净构建副本执行 `mvn -Dmaven.repo.local=/tmp/mall-m2 -Dmaven.test.skip=true package -pl mall-admin -am` 通过，并已重建 `mall-admin`、`admin-web`。admin 登录后通过 `/api/brand/create`、`/api/brand/update/{id}`、`/api/brand/{id}`、`/api/brand/update/showStatus`、`/api/brand/update/factoryStatus`、`/api/brand/delete/{id}` 完成品牌创建、品牌副标题持久化、编辑、状态切换和删除；通过 `/api/productCategory/create`、`/api/productCategory/update/{id}`、`/api/productCategory/{id}`、`/api/productAttribute/category/list/withAttr`、`/api/productAttribute/attrInfo/{id}`、`/api/productCategory/update/showStatus`、`/api/productCategory/update/navStatus`、`/api/productCategory/delete/{id}` 完成分类创建、编辑、筛选属性关联、状态切换和删除；`/catalog/brands`、`/catalog/categories` 前端路由返回 HTTP 200；测试品牌和测试分类已清理。
- 2026-06-29 管理端商城用户/管理员迁移验证：`mall-admin-template` 执行 `npm run build` 通过；`mall-admin` 执行 `mvn -Dmaven.repo.local=<workspace>/.m2/repository -pl mall-admin -am -DskipTests compile` 和 `package` 通过，并已重建 `mall-admin`、`admin-web`。admin 登录后通过 `/api/mallUser/list`、`/api/mallUser/{id}`、`/api/mallUser/update/{id}`、`/api/mallUser/updateStatus/{id}`、`/api/mallUser/batchFreeze`、`/api/mallUser/delete/{id}` 完成商城会员列表、详情、资料更新、冻结、批量冻结和删除；通过 `/api/admin/register`、`/api/admin/list`、`/api/admin/update/{id}`、`/api/admin/updateStatus/{id}`、`/api/admin/role/update`、`/api/admin/role/{id}`、`/api/admin/delete/{id}` 完成管理员新增、列表、编辑、启停、角色分配、角色清空和删除；`/system/admins` 前端路由返回 SPA HTML；测试会员和测试管理员已清理。
- 2026-06-29 管理端轮播图/系统设置迁移验证：`mall-admin-template` 执行 `npm run build` 通过；`mall-admin` 执行 `mvn -Dmaven.repo.local=<workspace>/.m2/repository -pl mall-admin -am -DskipTests compile` 和 `package` 通过，并已重建 `mall-admin`、`admin-web`。admin 登录后通过 `/api/banner/create`、`/api/banner/list`、`/api/banner/{id}`、`/api/banner/update/{id}`、`/api/banner/updateStatus/{id}`、`DELETE /api/banner/delete/{id}` 完成轮播图创建、列表、详情、编辑、启停和删除；通过 `/api/sysSetting/create`、`/api/sysSetting/list`、`/api/sysSetting/{id}`、`/api/sysSetting/getValue/{key}`、`/api/sysSetting/update/{id}`、`/api/sysSetting/updateStatus`、`/api/sysSetting/getAllSettings`、`/api/sysSetting/updateBatch`、`/api/sysSetting/delete/batch`、`DELETE /api/sysSetting/delete/{id}` 完成系统设置创建、列表、详情、取值、编辑、启停、获取全部启用设置、批量更新、批量删除和删除；`/marketing/banners`、`/system/settings` 前端路由返回 SPA HTML；测试轮播图和测试系统设置已清理。
- 2026-06-29 用户端账户设置/密码/邮箱验证迁移验证：`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过；`mall-portal` 执行 `mvn -Dmaven.repo.local=<workspace>/.m2/repository -pl mall-portal -am -DskipTests compile` 和 `package` 通过，并已重建 `mall-portal`、`user-web`。`/account/settings`、`/auth/forgot-password`、`/auth/reset-password?token=fake`、`/auth/email-confirm?token=fake` 前端路由返回 HTTP 200；临时会员完成 `/portal/sso/register/json`、`/portal/sso/login`、`/portal/sso/updateInfo`、`/portal/sso/info`、`/portal/sso/updatePassword`、Redis 模拟 `/portal/sso/resetPassword/confirm`、Redis 模拟 `/portal/sso/email/confirm` 链路；修改密码后新密码可登录，重置密码后重置密码可登录，邮箱确认后 `email_verified=1`；测试会员和 Redis token 已清理。
- 2026-06-29 用户端确认下单迁移验证：`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过，并已重建 `user-web`；`/checkout`、`/cart` 前端路由返回 HTTP 200，`/portal/home/content` 代理返回业务 `code: 200`。临时会员完成 `/portal/sso/register/json`、`/portal/sso/login`、`/portal/member/address/add`、`/portal/member/address/list`、`/portal/cart/add`、`/portal/cart/list`、`/portal/order/generateConfirmOrder`、`/portal/order/generateOrder`、`/portal/order/detail/{orderId}` 链路；确认单返回商品与收货地址，订单状态为 `0` 待付款；测试会员、地址、购物车和订单数据已清理。
- 2026-06-29 用户端品牌/新热商品入口迁移验证：`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过，并已重建 `user-web`；`/brands`、`/products/new`、`/products/hot` 前端路由返回 HTTP 200；`/portal/brand/recommendList?pageNum=1&pageSize=30` 和 `/portal/product/list?pageNum=1&pageSize=3&sort=create_time&order=desc` 均返回业务 `code: 200`。
- 2026-06-29 用户端基础内容页迁移验证：`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过，并已重建 `user-web`；`/about`、`/contact`、`/help`、`/notice` 前端路由返回 HTTP 200，`mall-portal` 重启后启动完成。
- 2026-06-29 本地自确认支付验证：`mall-portal` 执行 `mvn -Dmaven.repo.local=<workspace>/.m2/repository -pl mall-portal -am -DskipTests package` 通过；`mall-backend-template/deployment/mall-portal.jar` 与 target jar SHA 一致，已重建 `mall-portal` 容器并启动完成。当前运行库补齐 `STRIPE_CHECKOUT` 支付方式；临时会员完成注册、登录、收货地址、购物车、`/portal/order/generateOrder`、`/portal/payment/h5/methods`、`/portal/payment/h5/create`、`/portal/payment/h5/status/{orderSn}/STRIPE_CHECKOUT`、`/portal/order/detail/{orderId}` 链路；创建支付返回 `provider=LOCAL_SELF`、`paymentStatus=SUCCESS`、`redirectRequired=false`，订单状态进入 `1` 待发货；测试会员/订单/支付记录已清理，商品 1 的商品库存/销量和 SKU 库存/锁定库存恢复到测试前快照。
- 2026-06-29 用户端旧页面缺口审计验证：对照 `legacy-uniapp/src/pages.json` 的正式页面清单补齐新 Vue 3 页面或轻量替代入口；`mall-user-template` 执行 `npm run typecheck` 和 `npm run build` 通过，并已重建 `user-web`。`/`、`/products`、`/products/1`、`/brands`、`/brands/1`、`/categories`、`/cart`、`/checkout`、`/orders`、`/orders/1/pay`、`/account`、`/account/settings`、`/account/addresses`、`/account/history`、`/account/collections`、`/account/brand-attentions`、`/about`、`/contact`、`/help`、`/notice`、`/auth`、`/auth/forgot-password`、`/auth/reset-password?token=fake`、`/auth/email-confirm?token=fake` 前端路由均返回 HTTP 200；`/portal/home/content` 代理返回业务 `code: 200`。2026-06-30 起运行层已移除旧 UniApp hash 兼容。
- 2026-06-29 前台/admin 英文优先验证：`mall-user-template` 和 `mall-admin-template` 均执行 `npm run typecheck`、`npm run build` 通过，并已重建 `user-web`、`admin-web`。`src/i18n.ts` 在两个前端保持一致，默认 locale 逻辑为无本地选择时返回 `en`，`setLocale` 会写入 `mall.locale` 并更新 `document.documentElement.lang`；构建产物包含 `mall.locale`、`MutationObserver`、`Home`、`Dashboard`、`Chinese` 等 locale 运行时证据。路由烟测 `/`、`/products`、`/account/settings`、`/orders/1/pay`、`/login`、`/dashboard`、`/catalog/products`、`/system/admins` 均返回 HTTP 200。尝试用 Playwright 做真实浏览器快照，但当前机器没有 Chrome/Chromium；`install-browser chrome` 需要交互式 sudo，`install-browser chromium` 不支持 ubuntu26.04-x64，因此本轮未完成浏览器截图级验证。
- 2026-06-30 轻量模板收尾验证：`mall-user-template` 和 `mall-admin-template` 均执行 `npm run typecheck`、`npm run build` 通过；PostgreSQL 初始化 SQL 在临时库 `mall_template_check_20260630` 完整执行成功并输出 `Mall template database initialization complete.`；已重建 `user-web`、`admin-web`、`mall-admin`、`mall-portal`。当前 6 个 Compose 服务均 `Up`，PostgreSQL/Redis healthy；`/favicon.png` 和 `/template-assets/product-1.svg` HTTP 200 且容器权限为 `644`；`/portal/brand/detail/1` 返回 `Nova Supply` 和本地 `/template-assets/brand-1.svg`；`/portal/product/list?pageNum=1&pageSize=3` 返回 3 个英文商品和本地图片路径；`demo_user / 123456`、`demo_viewer / 123456` 登录均返回业务 `code: 200`；`demo_viewer` 仅拥有 catalog/banner/settings/email/dashboard/admin info/logout 的只读资源；`WebLogAspect` 已脱敏 `password/token/secret/credential` 字段，最新登录日志显示 `password:"******"`。

## 注意事项

- `mall-backend-template/deployment/mall-admin.jar` 和 `mall-portal.jar` 是本地部署构建产物，不应提交为模板源码的一部分。
- 新 clone 或清理后需要先构建 jar，再执行 compose。
- `mall-template-docs/deploy/local-docker/.env.example` 是默认本地配置模板；生产环境应复制为 `.env` 并修改密码、域名、SMTP 和支付参数。
- 当前模板保留 Redis，因为认证和缓存路径仍依赖它；相比 MinIO/ES/MQ/Mongo，Redis 是轻量且直接参与运行链路的依赖。
