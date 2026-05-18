# Trump Truth Archive Clone - Astro Version

这是一个使用 Astro 框架复刻的特朗普真相档案库网站，可部署在 Cloudflare Pages。

## 项目特性

- **高度一致的设计**：完全复刻原网站的视觉设计和布局
- **实时数据更新**：支持从 API 获取最新发言数据
- **中英文切换**：支持中文和原文两种语言显示
- **响应式设计**：完美适配桌面、平板和手机设备
- **静态生成**：使用 Astro 生成纯静态 HTML，无需服务器
- **Cloudflare Pages 部署**：一键部署到 Cloudflare Pages

## 项目结构

```
trump-truth-astro/
├── src/
│   ├── layouts/
│   │   └── Layout.astro          # 主布局组件
│   └── pages/
│       └── index.astro           # 首页
├── astro.config.mjs              # Astro 配置
├── tsconfig.json                 # TypeScript 配置
├── package.json                  # 项目依赖
├── wrangler.toml                 # Cloudflare Workers 配置
└── README.md                     # 本文件
```

## 安装与开发

### 1. 安装依赖

```bash
cd trump-truth-astro
pnpm install
```

### 2. 本地开发

```bash
pnpm run dev
```

访问 `http://localhost:3000` 查看网站。

### 3. 构建生产版本

```bash
pnpm run build
```

构建后的文件位于 `dist/` 目录。

## 配置 API 数据源

默认情况下，网站尝试从 `/api/pulse/trump-truth-archive` 获取数据。您可以：

1. **使用原网站的 API**（需要配置 CORS 代理）
2. **创建自己的 API 端点**
3. **使用静态数据**（编辑 `src/pages/index.astro` 中的 `samplePosts`）

### 示例 API 响应格式

```json
{
  "ok": true,
  "posts": [
    {
      "id": "post-id",
      "created_at": "2026-05-18T13:00:00Z",
      "created_at_text": "1 小时前",
      "content": "English content here",
      "content_zh_cn": "中文内容在这里",
      "avatar": "https://example.com/avatar.jpg",
      "url": "https://truthsocial.com/@realDonaldTrump/123",
      "media": [
        {
          "type": "image",
          "url": "https://example.com/image.jpg"
        }
      ]
    }
  ]
}
```

## 部署到 Cloudflare Pages

### 方法 1：使用 Git 连接

1. 将项目推送到 GitHub/GitLab
2. 登录 Cloudflare Dashboard
3. 进入 Pages 部分
4. 选择 "Connect a Git repository"
5. 选择您的仓库
6. 配置构建设置：
   - **Framework preset**: None
   - **Build command**: `pnpm install && pnpm run build`
   - **Build output directory**: `dist`
7. 点击 "Save and Deploy"

### 方法 2：使用 Wrangler CLI

```bash
# 安装 Wrangler
pnpm add -D wrangler

# 登录 Cloudflare
pnpm wrangler login

# 部署
pnpm wrangler pages deploy dist
```

### 方法 3：手动上传

1. 运行 `pnpm run build`
2. 在 Cloudflare Pages 中上传 `dist/` 文件夹

## 自定义

### 修改导航链接

编辑 `src/pages/index.astro` 中的导航部分：

```astro
<a href="/" class="navchip">⌂ 首页</a>
<a href="/korea-tech" class="navchip">◉ 韩股科技</a>
<a href="/morning-report" class="navchip">✦ 今晨晨报</a>
```

### 修改样式

所有样式都在 `src/layouts/Layout.astro` 的 `<style is:global>` 标签中。您可以直接修改 CSS 变量：

```css
:root {
  --bg: #f5f7fb;
  --accent: #2563eb;
  /* ... 其他变量 */
}
```

### 添加新页面

在 `src/pages/` 目录中创建新的 `.astro` 文件，例如 `korea-tech.astro`：

```astro
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="韩股科技">
  <!-- 页面内容 -->
</Layout>
```

## 性能优化

- **静态生成**：所有页面都是静态 HTML，加载速度极快
- **图片优化**：使用 `loading="lazy"` 延迟加载图片
- **缓存策略**：Cloudflare Pages 自动缓存所有静态资源
- **API 缓存**：前端脚本中设置了 `cache: 'no-store'` 以获取最新数据

## 浏览器兼容性

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- 移动浏览器（iOS Safari, Chrome Mobile）

## 故障排除

### 数据不加载

1. 检查浏览器控制台是否有错误
2. 验证 API 端点是否可访问
3. 检查 CORS 设置（如果使用外部 API）
4. 确保 API 响应格式正确

### 样式不正确

1. 清除浏览器缓存
2. 运行 `pnpm run build` 重新构建
3. 检查 CSS 变量是否正确定义

### 部署失败

1. 检查构建日志
2. 确保 `dist/` 目录存在
3. 验证 `package.json` 中的脚本命令

## 许可证

MIT

## 支持

如有问题，请提交 Issue 或 Pull Request。

## 更新日志

### v1.0.0 (2026-05-18)
- 初始版本发布
- 完整复刻原网站设计
- 支持 Cloudflare Pages 部署
- 中英文切换功能
- 响应式设计
