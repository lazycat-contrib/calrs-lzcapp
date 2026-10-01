# calrs-lzcapp

[calrs](https://github.com/olivierlambert/calrs) 的懒猫微服打包（双商店）：自托管的预约调度平台，类似 Cal.com，Rust 写的。

- **镜像**：`ghcr.io/olivierlambert/calrs`（上游自带 amd64/arm64）。官方商店交付走 `delivery.mode: lazycat`，由 Action 转存到懒猫镜像源并改写 manifest。
- **路由**：`/` → `calrs:3000`。
- **`public_path: [/]`**：预约页本来就是给外部访客看的（访客挑时间、拿 `.ics`）；后台与设置由应用自己的账号体系保护——**第一个注册的账号是管理员**。
- **`user: root`**：镜像里是非 root 的 calrs 用户，而 `/lzcapp/var/calrs` 由平台以 root 创建，用 root 起才能写数据目录（上游给出的 compose 也是这么写的）。
- **环境**：
  - `CALRS_BASE_URL=https://<应用域名>`：预约链接、邮件、`.ics`、分享图都用它；
  - `CALRS_SECRET_KEY` **不设**：它要求 base64 的 32 字节，模板生成的随机串不满足会让容器起不来。交给 calrs 首次启动时自动生成，落在数据目录里随卷持久化（清空数据目录会让已存的 CalDAV/SMTP 密码解不开）。
- **持久化**：`/lzcapp/var/calrs:/var/lib/calrs`（SQLite 数据库、模板、凭据密文）。
- **图标**：取上游 `assets/calrs.png`（日历螃蟹）铺在深色圆角底上，压到 8bit 以满足官方商店 <200KiB 的要求。

## 截图

`.github/screenshots/` 里是本地跑起 1.18.0（把镜像层解开、直接运行里面的二进制）后用无头 Chromium 实拍的：

- `pc-*.png` 1600×900（16:9）：Dashboard、Event types、Bookings、公开预约页、Admin；
- `mobile-*.png` 1280×720：390×844 手机视口实拍，再居中放到模糊底图上——官方商店会把每张图按 16:9 居中裁切，直接传竖屏图会被裁掉大半。

## 已知取舍

- 没有配置容器健康检查：上游没有健康端点，镜像里也没有 curl/wget；微服入口自己的探活足够（单服务、无依赖门）。
- 邮件默认未配置：预约确认/提醒需要先在 Admin → SMTP 里填好（未配置时预约会停在待确认状态）。
