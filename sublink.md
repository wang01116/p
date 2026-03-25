# ProxyGate — 部署指南

通过 Cloudflare Workers + GitHub Pages 实现纯前端代理浏览外网。

---

## 架构概述

```
用户浏览器 (GitHub Pages)
    ↓ HTTPS
Cloudflare Worker (cloudflare-worker.js)
    ↓ 代理请求 + HTML 改写
目标外网站点
```

---

## 第一步：部署 Cloudflare Workers 后端

### 方法 A：Dashboard 手动部署（推荐新手）

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 左侧菜单 → **Workers & Pages** → **Create application** → **Create Worker**
3. 给 Worker 命名（如 `proxygate`），点击 **Deploy**
4. 点击 **Edit code**，将 `cloudflare-worker.js` 的全部内容粘贴进去
5. 点击右上角 **Save and deploy**
6. 记录 Worker URL，格式如：`https://proxygate.yourname.workers.dev`

### 方法 B：使用 Wrangler CLI

```bash
npm install -g wrangler
wrangler login
wrangler deploy cloudflare-worker.js --name proxygate --compatibility-date 2024-01-01
```

### Worker 路由说明

| 路径 | 作用 |
|------|------|
| `GET /` | 健康检查，返回 `{"status":"ok"}` |
| `POST /api/parse-node` | 解析单条节点字符串 |
| `GET /api/speed-test?host=HOST&port=PORT` | 测量到节点的延迟 |
| `GET /proxy?url=TARGET` | 代理目标 URL 并改写 HTML |

---

## 第二步：部署 GitHub Pages 前端

1. 在 GitHub 创建新仓库（如 `proxygate`）
2. 将 `index.html` 上传到仓库根目录
3. 进入仓库 **Settings** → **Pages**
4. Source 选择 `main` 分支，目录选 `/ (root)`
5. 点击 **Save**，等待 1～2 分钟
6. 访问 `https://你的用户名.github.io/proxygate/`

---

## 第三步：使用

1. 打开 GitHub Pages 地址
2. 在输入框粘贴 Worker URL（如 `https://proxygate.yourname.workers.dev`）
3. 点击**连接**，验证成功后进入主界面
4. 在左侧侧边栏粘贴节点（每行一个），点击**解析**
5. 点击节点卡片选中节点
6. 在顶部地址栏输入网址或搜索词，点击**前往**

---

## 支持的节点协议

| 协议 | 前缀 |
|------|------|
| VMess | `vmess://` |
| VLESS | `vless://` |
| Shadowsocks | `ss://` |
| Trojan | `trojan://` |
| Hysteria2 | `hysteria2://` 或 `hy2://` |
| TUIC | `tuic://` |

---

## 功能说明

### 节点解析
- 自动识别协议、主机、端口、加密方式
- 节点名称乱码修复（UTF-8 + URL 解码）
- 根据节点名称自动识别国家并显示国旗 emoji

### 测速
- 通过 Worker 对节点主机发起 HTTP HEAD 请求测量往返延迟
- 绿色 < 200ms，黄色 < 500ms，红色 ≥ 500ms
- 支持单节点测速和全部测速

### 代理浏览
- HTML 页面中的链接、图片、CSS、JS 均经过 URL 改写
- 注入 JS 拦截 `fetch`、`XHR`、点击事件，确保内页跳转正常
- 支持文本、图片等富文本内容渲染

### 预设快捷网站
- DuckDuckGo（默认搜索引擎，跳转限制少）
- YouTube / YouTube Music（搜索和卡片展示）
- Nitter（X/Twitter 替代，无需登录）
- Reddit（old.reddit.com，文本兼容性好）
- GitHub（含 Release 页面）
- BBC / Economist（新闻文章）
- Telegram（telegram.org 频道公开页）
- Spotify / 巴哈姆特动漫疯（搜索和卡片）

---

## 已知限制

| 问题 | 原因 | 建议 |
|------|------|------|
| YouTube 视频无法播放 | 视频流需要特殊协议处理，Workers 无法转发媒体流 | 仅使用搜索功能 |
| Google Search 重定向失败 | Google 设置严格 COOP/CORP 头 | 改用 DuckDuckGo |
| 需要登录的页面无法保持会话 | Cookie 跨域限制 | 建议使用不需登录的替代站（如 Nitter） |
| 部分 SPA（单页应用）功能受限 | JS 动态路由无法完全改写 | 已注入 fetch/XHR 拦截器，尽力支持 |
| HTTPS 混合内容 | 部分站点 CDN 请求被浏览器拦截 | 暂无解决方案 |

---

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl/Cmd + K` | 聚焦地址栏 |
| `Ctrl/Cmd + B` | 展开/收起侧边栏 |
| `Escape` | 关闭当前代理页面 |

---

## 隐私说明

- 所有流量经由你自己的 Cloudflare Worker 中转，不经过任何第三方服务器
- Worker URL 存储在本地浏览器 localStorage，不上传
- 节点信息仅在内存中处理，刷新页面后清空

---

## 免责声明

本项目为技术研究用途，作者不对使用者的行为负责。请遵守当地法律法规。
