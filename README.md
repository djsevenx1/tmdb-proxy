# TMDB + Bangumi + GitHub 代理 Worker

> 🔱 **Fork 自 [HuntzzZ/tmdb-proxy](https://github.com/HuntzzZ/tmdb-proxy)** —— 致谢原作者。
> 本 fork 在原版基础上扩展了 **Bangumi (bangumi.tv / bgm.tv)** 代理能力, 一个 Worker 同时代理 TMDB、Bangumi 和 **GitHub Releases**。
> 📱 **作为 [LunaTV-Mobile](https://github.com/djsevenx1/LunaTV-Mobile) v2.1.46+ 的官方 backend** — 配套 worker URL 输到 App 「TMDB / Bangumi 代理 URL」+「GitHub 代理 URL」, 一键加速 TMDB + Bangumi + GitHub (检查更新 / APK 下载)。

一个基于 Cloudflare Workers 的 API 代理服务, 用于解决影视库 / 弹幕网刮削工具的跨域访问问题, **并配套 App 内建更新器走 worker 拉 GitHub release**。支持完整的 TMDB API + 图片代理, 新增 Bangumi API + 图片代理, **v2.1.46 新增 GitHub Releases API + release assets 流式下载代理**。

部署在 Cloudflare Pages (识别 `_worker.js`), 不是 Cloudflare Workers 经典模式。

## ✨ 功能特性

- 🔄 **完整 TMDB API 代理** (path-based, 不套娃): `/movie/{id}` `/tv/{id}` `/search/movie?query=xxx` 等
- 🖼️ **TMDB 图片代理**: `/image/t/p/{size}/{file}.jpg` → `image.tmdb.org/t/p/...`, 1 天 CDN cache
- 🎌 **Bangumi API 代理** *(本 fork 新增)*: `/bangumi/{path}` → `api.bgm.tv`, 透传客户端 `Authorization`, 强制 `LunaTV-Mobile/1.0` UA
- 🖼️ **Bangumi 图片代理** *(本 fork 新增)*: `/bgm-img/{path}` → `lain.bgm.tv`, 自动加 `Referer: https://bgm.tv/` 绕过反盗链
- 🔑 **`?api_key=` 客户端透传** (v2.1.40.1+): 不想在 Cloudflare Dashboard 配 `TMDB_API_KEY` env? 客户端请求带 `?api_key=xxx` 直接用
- 🌐 **CORS 支持**: 完整解决浏览器跨域问题
- 🏠 **主页**: 根路径 `/` 返 LunaTV 风的 HTML 主页, 跟 worker 状态 / 路由 / 致谢一目了然
- ⚡ **全球加速**: 基于 Cloudflare 全球边缘网络
- 💾 **智能缓存**: 图片 1 天缓存, 减少 API 调用

## 🚀 快速部署

### 前置要求

- [x] Cloudflare 账户 (免费版够用, 100k requests/day)
- [x] GitHub 账户 (用 Actions 自动部署) 或 Node.js 16+ (手动部署)
- [x] *(可选)* TMDB API 密钥 ([申请地址](https://www.themoviedb.org/settings/api)) — 不配也能用, 见 `?api_key=` 客户端透传
- [x] *(可选)* Bangumi access_token, 配 `BGM_ACCESS_TOKEN` env 后服务端注入, 客户端无需自带
- [x] *(可选)* GitHub PAT, 配 `GITHUB_TOKEN` env 后拉高 60/hr → 5000/hr 速率限制. 不配走匿名 60/hr, 检查更新够用

### 方式一: GitHub Actions 自动部署到 Cloudflare Pages (推荐)

> 推到 main 自动 build & deploy, 拿 `https://<project>.pages.dev` 即可。

1. **Fork 本仓库**
2. **配置 GitHub Secrets** (仓库 Settings → Secrets and variables → Actions):
   - `CLOUDFLARE_API_TOKEN`: Cloudflare API 令牌 (My Profile → API Tokens → Create Token → Edit Cloudflare Pages 模板)
   - `CLOUDFLARE_ACCOUNT_ID`: Cloudflare 账户 ID (Workers & Pages 详情页右侧栏可查)
   - *(可选)* `TMDB_API_KEY`: 服务端持有的 TMDB API Key. 不配也行, 见 `?api_key=` 透传
   - *(可选)* `Bgm_ACCESS_TOKEN`: 服务端持有的 Bangumi access_token
   - *(可选)* `GITHUB_TOKEN`: GitHub Personal Access Token, 拉高 60/hr → 5000/hr 速率限制 (v2.1.46+ GitHub 路由用)
3. **配置 Pages 项目名**: 编辑 `.github/workflows/deploy.yml` 里的 `project-name=tmdb-proxy` 改成你想要的
4. **推送代码到 main 分支自动部署**

### 方式二: Cloudflare Dashboard 上传 (一次性手动)

> 适合不想配 GitHub Actions / 只想试一下的用户。

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages** → **Create application** → **Pages** → **Upload assets** (直接上传)
2. **Project name**: 自取, 如 `tmdb-proxy`
3. 拖入整个项目根目录 (含 `_worker.js` / `worker.js` / `wrangler.toml`) → **Deploy site**
4. 部署完 → **Settings** → **Environment variables** 配置 (可选) `TMDB_API_KEY` / `Bgm_ACCESS_TOKEN` / `GITHUB_TOKEN`
5. 访问 `https://<project>.pages.dev/` 看 LunaTV 风主页

> ⚠️ **重要**: Cloudflare Pages 识别 Worker 用 **`_worker.js`** (下划线开头) 约定文件, 不是 `worker.js`。
> 本 repo 两个文件都保留 (内容一样), 既兼容 Pages 也兼容经典 Workers 部署。

### 方式三: 本地 wrangler 命令行

```bash
# 克隆项目
git clone https://github.com/djsevenx1/tmdb-proxy.git
cd tmdb-proxy
# 安装 wrangler
npm install -g wrangler
# 登录
wrangler login
# 部署到 Cloudflare Pages (推荐)
wrangler pages deploy . --project-name=tmdb-proxy
# 部署到 Cloudflare Workers 经典模式 (旧)
wrangler deploy
```

## 📖 使用方法

### 基础 URL

部署成功后，您可以在 worker 处设置自定义域，若保持默认，您的 Worker 地址为：
```
https://your-worker-name.your-subdomain.workers.dev
```

## 🎬 TMDB 代理 (原版功能)

### API 代理示例

**获取电影信息**
```
GET /movie/550
```

**搜索电影**
```
GET /search/movie?query=avatar
```

**获取电视剧信息**
```
GET /tv/1399
```

### 图片代理示例

**海报图片**
```
GET /image/t/p/w500/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg
```

**背景图片**
```
GET /image/t/p/original/hZkgoQYus5vegHoetLkCJzb17zJ.jpg
```

**简化路径**
```
GET /image/w500/jSziioSwPVrOy9Yow3XhWIBDjq1.jpg
```

## 🎌 Bangumi 代理 (本 fork 新增)

> 所有 Bangumi 路由都会自动:
> - 强制注入 `User-Agent: LunaTV-Mobile/1.0 (https://github.com/djsevenx1/LunaTV-Mobile)` (api.bgm.tv 强校验)
> - 透传客户端 `Authorization` header (如有); 若 Worker 配了 `BGM_ACCESS_TOKEN` env 则缺省用服务端 token
> - 加 `Referer: https://bgm.tv/` 头 (lain.bgm.tv 强校验)

### API 代理示例

**获取条目详情**
```
GET /bangumi/v0/subjects/1
```

**搜索条目**
```
GET /bangumi/v0/search/subjects?keyword=CLANNAD&type=2
```

**获取剧集列表**
```
GET /bangumi/v0/episodes?subject_id=1&type=0
```

**获取用户收藏**
```
GET /bangumi/v0/users/{username}/collections?subject_type=2
```

**获取每日放送**
```
GET /bangumi/calendar
```

### 图片代理示例

**封面图**
```
GET /bgm-img/r/400/pic/cover/l/xx/1/1.jpg
```

**角色头像**
```
GET /bgm-img/r/200/char_id/123.jpg
```

> 路径格式: `/bgm-img/{lain.bgm.tv 上的完整路径}`, 任意 `lain.bgm.tv` 下的资源都可代理。

## 🐙 GitHub 代理 (v2.1.46+ 新增)

> 配套 [LunaTV-Mobile](https://github.com/djsevenx1/LunaTV-Mobile) v2.1.46+ **app 内建更新器** (检查更新 + APK 下载), 解决国内 GFW 完全拉不到 `api.github.com` / `objects.githubusercontent.com` 的问题。
> 走 worker 反代 + CORS 头, App 端无感知, 进度条 / 取消 / 重试 / 调起 APK 安装器 都在 App 内完成, **不跳浏览器, 不用第三方 pub package**。

### Releases API 代理示例

**获取最新 release** (app 检查更新用):
```
GET /github/repos/djsevenx1/LunaTV-Mobile/releases/latest
```

**获取所有 releases**:
```
GET /github/repos/{owner}/{repo}/releases
```

> 任意 path-based 调用: `/github/repos/{owner}/{repo}/{path...}` → `https://api.github.com/repos/{owner}/{repo}/{path...}`
> Worker 自动加 `Accept: application/vnd.github.v3+json` + `User-Agent` (强制, GitHub API 拒空 UA). 配 `GITHUB_TOKEN` env 后自动注入 `Authorization` Bearer, 拉高 60/hr → 5000/hr 速率限制.

### Release asset 下载代理示例

**下载 APK** (app 内建下载器拿 APK 用):
```
GET /github/asset/djsevenx1/LunaTV-Mobile/v2.1.46/app-arm64-v8a-release.apk
```

> 格式: `/github/asset/{owner}/{repo}/{tag}/{asset_name}`
> Worker 内部转成 `https://github.com/{owner}/{repo}/releases/download/{tag}/{asset_name}` 跟 302 跳到 `objects.githubusercontent.com` 流式转发. **不 buffer 整 APK 到内存**, 几十 MB APK 直接 stream, 跟用户在代理浏览器下效果一样, 但 app 内可以画进度条.

## 🏠 主页 (Homepage)

> 根路径 `/` 返 LunaTV 风的 HTML 主页, 跟 worker 状态 / 路由表 / 致谢一目了然, 不用再手敲 `/health` 验证。

部署完访问 `https://<project>.pages.dev/` 直接看:

- ✅ Worker 状态 (在线 / 离线)
- 📋 路由表 (TMDB / Bangumi / 图片代理 4 大类)
- 🏷️ Fork 致谢 (HuntzzZ/tmdb-proxy)
- 🔗 配套项目链接 (LunaTV-Mobile 等)

`/health` 端点也保留, 返纯文本 `OK`, 适合 CI 健康检查。

## 📱 LunaTV-Mobile 集成 (v2.1.46+)

> 🎯 **本 repo 是 [LunaTV-Mobile](https://github.com/djsevenx1/LunaTV-Mobile) v2.1.46+ 的官方 backend**。

部署完本 worker, 拿到的 `https://<project>.pages.dev` 粘到 LunaTV-Mobile App 的:
**设置 → 数据源 → TMDB / Bangumi 代理 URL** + **设置 → 数据源 → GitHub 代理 URL**

配了后, App 内:
- **TMDB 数据源 / 图片源** → 走 `${workerUrl}/movie/...` + `${workerUrl}/image/...`
- **Bangumi 数据源 / 图片源** → 走 `${workerUrl}/bangumi/...` + `${workerUrl}/bgm-img/...`
- **检查更新 + APK 下载** (v2.1.46+ 内建下载器) → 走 `${workerUrl}/github/repos/.../releases/latest` + `${workerUrl}/github/asset/.../releases/download/...`
- **TMDB API Key** → 直接从 App 读, worker 端 `?api_key=` 透传, 不用去 Cloudflare Dashboard 配 env

一个 worker 同时加速 TMDB + Bangumi + GitHub 三套数据, 解决国内 GFW (TMDB 没 GFW 但网络慢, Bangumi 偶尔抽风, GitHub 100% 拉不到). 推荐填同一个 worker URL 即可.

## 🔧 刮削工具配置

### Jellyfin

1. 进入 **控制台** → **插件** → **TheMovieDb**
2. 配置：
   - API 地址：`https://您的worker.workers.dev`
   - 图片地址：`https://您的worker.workers.dev/image`

### TinyMediaManager

1. **Settings** → **Movies** → **TheMovieDb**
2. 配置：
   - API URL：`https://您的worker.workers.dev`
   - 图片基础 URL：`https://您的worker.workers.dev/image`

### Emby

1. 进入 **管理** → **高级** → **神医助手插件** → **元数据增强**（请自行安装神医助手）
2. 修改 API 服务器地址为您的 Worker URL

### Plex

使用 [TMDBMetaDataAgent](https://github.com/ZeroQI/TMDBMetaDataAgent.bundle) 插件，配置代理地址。

### Bangumi 工具 (本 fork 新增)

将 `api.bgm.tv` 和 `lain.bgm.tv` 替换为 Worker 即可:
```
api.bgm.tv    →  https://您的worker.workers.dev/bangumi
lain.bgm.tv   →  https://您的worker.workers.dev/bgm-img
```

## ⚙️ 配置说明

### 环境变量

| 变量名 | 描述 | 必需 |
|--------|------|------|
| `TMDB_API_KEY` | TMDB API 密钥 | ✅ |
| `CLOUDFLARE_API_TOKEN` | Cloudflare API 令牌 | ✅ |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare 账户 ID | ✅ |
| `BGM_ACCESS_TOKEN` *(本 fork 新增)* | Bangumi access_token, 缺省时透传客户端 `Authorization` | ❌ |

### wrangler.toml 配置

```toml
name = "tmdb-proxy"
compatibility_date = "2024-01-01"
compatibility_flags = ["nodejs_compat"]
# ⚠️ Cloudflare Pages 项目用 _worker.js, 不用 main 字段
# main = "worker.js"   # 经典 Workers 模式才用, Pages 部署会忽略
# 自定义域名 (可选)
# routes = [
#   "tmdb.yourdomain.com/*"
# ]
```

> **Pages vs Workers 经典**:
> - **Pages** (推荐): 走 `_worker.js` 约定文件, 部署用 `wrangler pages deploy` 或 GitHub Actions
> - **Workers 经典**: 走 `main` 字段, 部署用 `wrangler deploy`
> - 本 repo 两个文件 (`_worker.js` + `worker.js`) 内容一样, 兼容两种模式

## 🛠️ 开发指南

### 本地开发

```bash
# 启动开发服务器
wrangler dev
# 监听模式
wrangler dev --live-reload
# 查看日志
wrangler tail
```

### 项目结构

```
tmdb-proxy/
├── _worker.js             # Cloudflare Pages Worker 主逻辑 (下划线开头, Pages 约定)
├── worker.js              # 同样内容, 兼容 Cloudflare Workers 经典模式
├── wrangler.toml          # 配置文件 (Pages 模式无 main 字段)
├── .github/
│   └── workflows/
│       └── deploy.yml     # 自动部署工作流 (Pages)
└── README.md              # 项目文档
```

## 🐛 故障排除

### 常见问题

**❌ 部署失败：权限错误**
```bash
# 检查令牌权限
wrangler whoami
```

**❌ TMDB API 返回 401 错误**
- 优先级: `env.TMDB_API_KEY` (服务端) > 客户端 `?api_key=xxx` (透传, v2.1.40.1+)
- 没配 `TMDB_API_KEY` env → 客户端必须带 `?api_key=xxx` 才能用, 例:
  ```bash
  curl "https://<project>.pages.dev/movie/550?api_key=YOUR_KEY"
  ```
- 配了 `TMDB_API_KEY` env → 客户端可不带, worker 自动注入
- 配了但还是 401 → 检查 env 值是否正确, 改了 env 后要手动 Retry deployment (Cloudflare Pages env 变更不会自动触发)

**❌ Bangumi API 返回 400 错误**
- 强制 UA 已在 Worker 内注入, 不应再出现; 如出现检查 `User-Agent` 是否被你的中间层覆盖
- 如配了 `BGM_ACCESS_TOKEN` 仍 401, 检查 token 是否过期

**❌ Bangumi 图片返回 403 错误**
- Worker 已自动加 `Referer: https://bgm.tv/`, 不应再被拦; 如出现检查中间层是否剥头

**❌ 图片无法加载**
- 检查图片代理路径格式
- 验证图片 URL 是否可公开访问

**❌ 速率限制错误**
- TMDB 限制：30-40 请求/10秒
- Bangumi 限制：详见 [api.bgm.tv 文档](https://bangumi.github.io/api/)
- GitHub 限制：匿名 60/hr, 配 `GITHUB_TOKEN` 后 5000/hr
- 建议添加缓存减少调用

### 日志查看

```bash
# 实时日志
wrangler tail
# 特定环境日志
wrangler tail --env production
```

## 🔄 工作流优化

部署工作流已优化，只在代码文件更改时触发：
```yaml
on:
  push:
    branches: [ main ]
    paths:
      - '_worker.js'
      - 'worker.js'
      - 'wrangler.toml'
      - 'package.json'
      - '.github/workflows/deploy.yml'
```
README 更新不会触发不必要的部署。

## 📊 监控和维护

### 性能监控

1. **Cloudflare Dashboard**：查看请求量、错误率
2. **TMDB 账户**：监控 API 使用情况
3. **GitHub Actions**：检查部署状态

### 维护建议

- 定期更新 TMDB API 密钥
- 监控 API 调用频率
- 更新 Worker 代码以兼容 API 变更

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！
1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/新功能`
3. 提交更改：`git commit -m '添加新功能'`
4. 推送分支：`git push origin feature/新功能`
5. 提交 Pull Request

## 📄 许可证 & 致谢

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

> **致谢 & Changelog**:
> - 本项目 fork 自 [HuntzzZ/tmdb-proxy](https://github.com/HuntzzZ/tmdb-proxy), 感谢原作者 [@HuntzzZ](https://github.com/HuntzzZ) 的优秀实现
> - 本 fork 在原作者授权 (MIT) 基础上扩展 Bangumi + GitHub 代理能力, 改动记录见 commit history
> - 作为 [LunaTV-Mobile](https://github.com/djsevenx1/LunaTV-Mobile) v2.1.46+ 的官方 backend, 配合 App 一键加速 TMDB + Bangumi + GitHub
> - **v2.1.46** 新增 GitHub Releases API + release assets 流式下载代理 (`/github/repos/.../releases/latest` + `/github/asset/{owner}/{repo}/{tag}/{asset}`), 配套 App 内建下载器

## ⚠️ 免责声明

本项目仅用于学习和研究目的，请遵守：
- [TMDB API 使用条款](https://www.themoviedb.org/documentation/api/terms-of-use)
- [Bangumi API 使用条款](https://bgm.tv/about/guideline)
- [GitHub API 使用条款](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service)
- Cloudflare Workers 服务条款
- 当地法律法规

## 🆘 获取帮助

- [提交 Issue](https://github.com/djsevenx1/tmdb-proxy/issues)
- [TMDB API 文档](https://developers.themoviedb.org/3)
- [Cloudflare Workers 文档](https://developers.cloudflare.com/workers/)

---

**如果这个项目对您有帮助，请给个 ⭐️ 支持一下！**
