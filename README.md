# TMDB + Bangumi 代理 Worker

> 🔱 **Fork 自 [HuntzzZ/tmdb-proxy](https://github.com/HuntzzZ/tmdb-proxy)** —— 致谢原作者。
> 本 fork 在原版基础上扩展了 **Bangumi (bangumi.tv / bgm.tv)** 代理能力, 一个 Worker 同时代理 TMDB 和 Bangumi。

一个基于 Cloudflare Workers 的 API 代理服务, 用于解决影视库 / 弹幕网刮削工具的跨域访问问题。支持完整的 TMDB API + 图片代理, **并新增 Bangumi API + Bangumi 图片代理**。

## ✨ 功能特性

- 🔄 **完整 TMDB API 代理**: 无缝代理所有 TMDB API 请求
- 🖼️ **TMDB 图片代理**: 代理 TMDB 图片资源, 解决图片无法加载问题
- 🎌 **Bangumi API 代理** *(本 fork 新增)*: 代理 `api.bgm.tv`, 透传客户端 `Authorization`
- 🖼️ **Bangumi 图片代理** *(本 fork 新增)*: 代理 `lain.bgm.tv` 图片, 自动补 `Referer`
- 🌐 **CORS 支持**: 完整解决浏览器跨域问题
- 🔒 **安全认证**: 服务端注入 TMDB API 密钥, 客户端无需自带
- ⚡ **全球加速**: 基于 Cloudflare 全球边缘网络
- 💾 **智能缓存**: 图片 1 天缓存, 减少 API 调用

## 🚀 快速部署

### 前置要求

- [x] Cloudflare 账户
- [x] TMDB API 密钥（[申请地址](https://www.themoviedb.org/settings/api)）
- [x] GitHub 账户
- [x] *(可选)* Bangumi access_token, 配 `BGM_ACCESS_TOKEN` env 后服务端注入, 客户端无需自带

### 一键部署

1. **Fork 本仓库**
2. **配置 GitHub Secrets**：
   - 进入仓库 Settings → Secrets and variables → Actions
   - 添加以下 Secrets：
     - `CLOUDFLARE_API_TOKEN`：Cloudflare API 令牌，请选择Cloudflare Workers 模板
     - `CLOUDFLARE_ACCOUNT_ID`：Cloudflare 账户 ID（可选）
     - `TMDB_API_KEY`：您的 TMDB API 密钥
     - `BGM_ACCESS_TOKEN` *(可选)*: Bangumi access_token, 配了之后客户端请求 Bangumi API 无需自带 token
3. **自动部署**：推送代码到 main 分支将自动触发部署

### 手动部署

```bash
# 克隆项目
git clone https://github.com/your-username/tmdb-proxy.git
cd tmdb-proxy
# 安装依赖
npm install -g wrangler
# 配置环境变量
export CLOUDFLARE_API_TOKEN="your-api-token"
export CLOUDFLARE_ACCOUNT_ID="your-account-id"
export TMDB_API_KEY="your-tmdb-api-key"
export BGM_ACCESS_TOKEN="your-bangumi-access-token"   # 可选
# 部署
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
main = "worker.js"
# 自定义域名（可选）
routes = [
  "tmdb.yourdomain.com/*"
]
# 环境变量（通过 GitHub Secrets 设置）
[env.production.vars]
TMDB_API_KEY = "{{ secrets.TMDB_API_KEY }}"
BGM_ACCESS_TOKEN = "{{ secrets.BGM_ACCESS_TOKEN }}"   # 可选
```

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
├── worker.js              # Worker 主逻辑 (TMDB + Bangumi 双路由)
├── wrangler.toml          # 配置文件
├── package.json           # 依赖配置（可选）
├── .github/
│   └── workflows/
│       └── deploy.yml     # 自动部署工作流
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
- 检查 TMDB API 密钥是否正确
- 验证环境变量配置

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
      - 'worker.js'
      - 'wrangler.toml'
      - 'package.json'
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

> **致谢**: 本项目 fork 自 [HuntzzZ/tmdb-proxy](https://github.com/HuntzzZ/tmdb-proxy), 感谢原作者 [@HuntzzZ](https://github.com/HuntzzZ) 的优秀实现。
> 本 fork 在原作者授权 (MIT) 基础上扩展 Bangumi 代理能力, 改动记录见 commit history。

## ⚠️ 免责声明

本项目仅用于学习和研究目的，请遵守：
- [TMDB API 使用条款](https://www.themoviedb.org/documentation/api/terms-of-use)
- [Bangumi API 使用条款](https://bgm.tv/about/guideline)
- Cloudflare Workers 服务条款
- 当地法律法规

## 🆘 获取帮助

- [提交 Issue](https://github.com/djsevenx1/tmdb-proxy/issues)
- [TMDB API 文档](https://developers.themoviedb.org/3)
- [Cloudflare Workers 文档](https://developers.cloudflare.com/workers/)

---

**如果这个项目对您有帮助，请给个 ⭐️ 支持一下！**
