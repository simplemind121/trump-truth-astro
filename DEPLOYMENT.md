# Cloudflare Pages 部署指南

本文档详细说明如何将 Trump Truth Archive 克隆项目部署到 Cloudflare Pages。

## 前置要求

- Cloudflare 账户（免费账户即可）
- GitHub/GitLab 账户（用于连接 Git 仓库）
- Node.js 和 pnpm（本地开发用）

## 部署方法

### 方法 1：使用 Git 连接（推荐）

这是最简单的部署方式，支持自动部署。

#### 步骤 1：将项目推送到 GitHub

```bash
# 初始化 Git 仓库
git init
git add .
git commit -m "Initial commit: Trump Truth Archive clone with Astro"

# 添加远程仓库
git remote add origin https://github.com/YOUR_USERNAME/trump-truth-astro.git

# 推送到 GitHub
git branch -M main
git push -u origin main
```

#### 步骤 2：连接到 Cloudflare Pages

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 在左侧菜单中选择 **Pages**
3. 点击 **Connect a Git repository**
4. 选择 **GitHub** 并授权
5. 选择 `trump-truth-astro` 仓库
6. 点击 **Begin setup**

#### 步骤 3：配置构建设置

在"Build settings"页面中配置：

- **Framework preset**: None（或 Astro，如果可用）
- **Build command**: `pnpm install && pnpm run build`
- **Build output directory**: `dist`
- **Root directory**: `/`（默认）

#### 步骤 4：环境变量（可选）

如果需要 API 端点或其他配置，在"Environment variables"中添加：

```
API_ENDPOINT=https://your-api.example.com
```

#### 步骤 5：部署

1. 点击 **Save and Deploy**
2. Cloudflare 将自动构建并部署您的网站
3. 部署完成后，您将获得一个 `*.pages.dev` 域名

### 方法 2：使用 Wrangler CLI

如果您已安装 Wrangler，可以使用命令行部署。

#### 步骤 1：安装 Wrangler

```bash
pnpm add -D wrangler
```

#### 步骤 2：登录 Cloudflare

```bash
pnpm wrangler login
```

这将打开浏览器进行身份验证。

#### 步骤 3：部署

```bash
# 构建项目
pnpm run build

# 部署到 Pages
pnpm wrangler pages deploy dist
```

#### 步骤 4：获取部署 URL

部署完成后，您将获得一个 `*.pages.dev` 域名。

### 方法 3：手动上传

如果您不想使用 Git，可以手动上传文件。

#### 步骤 1：构建项目

```bash
pnpm run build
```

#### 步骤 2：上传到 Cloudflare Pages

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 选择 **Pages**
3. 点击 **Create a project** > **Direct upload**
4. 上传 `dist` 文件夹中的所有文件
5. 点击 **Deploy site**

## 配置自定义域名

部署后，您可以将自定义域名绑定到您的网站。

### 步骤 1：在 Cloudflare 中配置域名

1. 在 Cloudflare Dashboard 中选择您的 Pages 项目
2. 进入 **Settings** > **Custom domains**
3. 点击 **Add custom domain**
4. 输入您的域名（例如 `trump-archive.example.com`）

### 步骤 2：更新 DNS 记录

如果域名在 Cloudflare 中注册：
- Cloudflare 将自动配置 DNS 记录

如果域名在其他注册商：
- 添加 CNAME 记录，指向 `<project-name>.pages.dev`

## 环境变量和 API 配置

### 配置 API 端点

如果您想使用自定义 API 端点而不是默认的 `/api/pulse/trump-truth-archive`：

#### 方法 1：在 Cloudflare Pages 中设置环境变量

1. 进入 Pages 项目的 **Settings**
2. 选择 **Environment variables**
3. 添加变量：
   - **Variable name**: `PUBLIC_API_ENDPOINT`
   - **Value**: `https://your-api.example.com/api/pulse/trump-truth-archive`

#### 方法 2：在代码中配置

编辑 `src/pages/index.astro`，修改脚本部分：

```javascript
const FEED = import.meta.env.PUBLIC_API_ENDPOINT || '/api/pulse/trump-truth-archive';
```

### API 代理配置

如果原始 API 存在 CORS 问题，您可以使用 Cloudflare Workers 作为代理。

创建 `functions/api/proxy.js`：

```javascript
export async function onRequest(context) {
  const url = new URL(context.request.url);
  const targetUrl = 'https://hketf-lab.pages.dev' + url.pathname.replace('/api/proxy', '');
  
  const response = await fetch(targetUrl, {
    headers: {
      'User-Agent': context.request.headers.get('User-Agent'),
    }
  });
  
  return new Response(response.body, {
    status: response.status,
    headers: {
      'Content-Type': response.headers.get('Content-Type'),
      'Access-Control-Allow-Origin': '*',
    }
  });
}
```

## 故障排除

### 构建失败

**问题**：`pnpm: command not found`

**解决方案**：
1. 在 Cloudflare Pages 设置中添加环境变量：
   - **Name**: `NODE_VERSION`
   - **Value**: `22`
2. 或在构建命令中使用 `npm`：`npm install && npm run build`

### 页面不显示

**问题**：部署成功但页面为空

**解决方案**：
1. 检查浏览器控制台是否有错误
2. 验证 `dist` 目录中是否有 `index.html`
3. 检查 Cloudflare Pages 的构建日志

### API 数据不加载

**问题**：页面加载但没有数据

**解决方案**：
1. 检查 API 端点是否正确
2. 验证 CORS 设置
3. 在浏览器控制台检查网络请求
4. 考虑使用 CORS 代理或 Cloudflare Workers

## 自动部署

使用 Git 连接后，每次推送到 `main` 分支时，Cloudflare Pages 将自动构建和部署。

### 预览部署

对于 Pull Requests，Cloudflare Pages 会自动创建预览部署。这允许您在合并前测试更改。

## 性能优化

### 启用 Cloudflare 功能

1. **缓存**：在 Pages 项目中启用缓存以加快加载速度
2. **压缩**：启用 Brotli 压缩
3. **安全**：启用 DDoS 保护和 WAF

### 监控

1. 进入 Pages 项目的 **Analytics** 选项卡
2. 查看流量、错误率和性能指标

## 常见问题

**Q: 如何更新网站？**
A: 只需推送代码到 GitHub，Cloudflare Pages 将自动构建和部署。

**Q: 如何回滚到之前的版本？**
A: 在 Cloudflare Pages 中，选择 **Deployments** 选项卡，找到之前的部署，点击 **Rollback**。

**Q: 如何添加自定义 404 页面？**
A: 创建 `src/pages/404.astro` 文件，Astro 将自动使用它作为 404 页面。

**Q: 如何添加 SSL 证书？**
A: Cloudflare Pages 自动为所有网站提供 SSL 证书。

## 支持

- [Cloudflare Pages 文档](https://developers.cloudflare.com/pages/)
- [Astro 部署指南](https://docs.astro.build/en/guides/deploy/)
- [Cloudflare 社区论坛](https://community.cloudflare.com/)

## 下一步

部署后，您可以：

1. 配置自定义域名
2. 设置 API 端点
3. 添加更多页面和功能
4. 配置分析和监控
5. 优化性能
