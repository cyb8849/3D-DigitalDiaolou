# 部署与运维说明

## 站点信息

| 项 | 值 |
|---|---|
| 公网地址 | http://43.138.129.208/ |
| 实例 ID | lhins-q4ul237p（广州 · 2C2G · 50G SSD） |
| 系统 | OpenCloudOS 9.6 + nginx 1.30.6（宝塔） |
| 站点根目录 | `/www/wwwroot/diaolou` |
| nginx 配置 | `/www/server/panel/vhost/nginx/diaolou.conf` |
| 访问日志 | `/www/wwwlogs/diaolou.log` |
| 错误日志 | `/www/wwwlogs/diaolou.error.log` |

目录结构：`index.html`、`vendor/`（three.js 三个库）、`models/`（3 个 glb）、`assets/`（18 张 webp），共 25 个文件，约 35MB。

## nginx 关键配置

- `try_files $uri $uri/ /index.html`：页面走单页回退
- 静态资源单独 location：`try_files $uri =404`，缺失资源返回真 404（不会回退成 HTML）
- gzip：HTML / JS / CSS / JSON / SVG，three.js 从 589KB 压到 151KB
- 缓存：静态资源 30 天；`index.html` 明确 no-cache，保证更新即时生效
- `favicon.ico` 返回 204，避免回落到首页

## 如何更新内容

### 方式一：宝塔面板上传（最省事，适合改文案）
1. 宝塔面板 → 文件 → `/www/wwwroot/diaolou`
2. 直接编辑或上传覆盖，静态文件无需重启

### 方式二：git 拉取（适合多文件改动）
前提是本地已把改动推到 GitHub（`cyb8849/3D-DigitalDiaolou`）。

```bash
cd /www/wwwroot/diaolou
git init && git remote add origin https://github.com/cyb8849/3D-DigitalDiaolou.git
git fetch origin main && git checkout -f origin/main
```

注意：首次执行会覆盖服务器上的文件，确保远端已是最新版本。

### 方式三：临时中转（本机无法直连 GitHub 时）
1. 在 WorkBuddy 里临时发布项目，拿到带 `sharecode` 的链接
2. 在服务器上按文件逐个拉取：
   ```bash
   curl -sS -o /tmp/x.glb "https://<host>/models/x.glb?sharecode=<code>"
   ```
3. 拷进站点目录后，立即下线临时发布

> TAT 远程命令对「上传/接收」类操作有安全拦截（写脚本、自建监听、WebDAV PUT 均被拒），所以不能用它传文件，只能用服务器主动下载。

## 已验证

- 首页 200（116,445B）
- `models/*.glb` 三个文件与本地 **md5 完全一致**
- `vendor/*.js`、`assets/*.webp` 与本地 md5 一致
- gzip 生效：index 116KB → 32KB，three.js 589KB → 151KB
- 缺失静态资源返回 404，favicon 返回 204

## 已知限制

1. **无 HTTPS**：没有绑定域名，无法申请证书。接入域名后可一键开 Let's Encrypt。
2. **22MB 模型加载慢**：实测服务器出网约 4Mbps，`zhangbaijiashu.glb`（22.6MB）下载需 **51 秒**。建议按审计清单做纹理压缩到 5MB 以内。
3. **宝塔面板端口 41287 对全网开放**：建议在轻量云防火墙里限制为自己的固定 IP。
4. **8080 端口开放但无服务**（原 Spring Boot 用途）：不用可在防火墙关闭。

## 常用命令

```bash
nginx -t                          # 校验配置
nginx -s reload                   # 热加载
tail -f /www/wwwlogs/diaolou.log  # 看访问日志
du -sh /www/wwwroot/diaolou       # 看站点体积
```
