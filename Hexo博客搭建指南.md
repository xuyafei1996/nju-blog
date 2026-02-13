# Hexo 静态博客全方位搭建指南

本指南旨在提供一套通用的、现代化的 Hexo 博客搭建流程，融合了基础概念、环境配置、自动化部署（CI/CD）以及常见踩坑解决方案。

---

## 一、Hexo 概念与原理

**Hexo** 是一个快速、简洁且高效的博客框架。
*   **原理**：它基于 Node.js，将 Markdown 文档解析并渲染成静态网页（HTML/CSS/JS）。
*   **优势**：
    *   **极速**：静态页面加载速度快，利于 SEO。
    *   **免费**：可托管在 GitHub Pages、Vercel 等平台，无需服务器成本。
    *   **扩展**：拥有丰富的主题和插件生态。

---

## 二、环境准备 (Prerequisites)

在开始之前，请确保你的电脑已安装以下工具。

### 1. 核心工具
*   **Node.js** (运行环境)：建议安装 **LTS (长期支持版)**，如 Node 18 或 20。
    *   验证：`node -v`, `npm -v`
*   **Git** (版本控制)：用于管理代码和推送发布。
    *   验证：`git --version`

### 2. ⚡ 进阶配置（强烈推荐）

#### 2.1 使用 NVM 管理 Node 版本
为了避免版本冲突，建议使用 **NVM** (Node Version Manager) 管理 Node 版本。
*   **常用命令**：
    ```bash
    nvm install lts      # 安装最新 LTS
    nvm use lts          # 切换版本
    ```

#### 2.2 配置国内镜像源
解决 npm 下载慢或卡顿的问题：
```bash
# 永久配置淘宝镜像源
npm config set registry https://registry.npmmirror.com
```

---

## 三、安装与本地使用

### 1. 初始化项目
推荐使用 `npx` 运行 Hexo，无需全局安装，避免污染环境。

```bash
# 初始化博客目录
npx hexo-cli init my-blog

# 进入目录
cd my-blog
```

### 2. 安装依赖 (⚠️ 关键点)
由于 Hexo 部分插件依赖较旧，**必须**加上兼容参数，否则容易报错。

```bash
npm install --legacy-peer-deps
```

### 3. 本地预览与写作
*   **启动服务器**：
    ```bash
    npx hexo s
    # 访问 http://localhost:4000
    ```
*   **新建文章**：
    ```bash
    npx hexo new "文章标题"
    # 文件生成在 source/_posts/文章标题.md
    ```

---

## 四、GitHub 自动化部署 (GitHub Actions)

传统的 `hexo d` 本地部署方式容易因环境问题失败，**强烈推荐**使用 GitHub Actions 实现云端自动构建和发布。

### 1. 准备工作
1.  在 GitHub 创建一个仓库（公开库）。
2.  将本地代码推送到该仓库。

### 2. 配置自动化脚本
在项目根目录创建文件：`.github/workflows/deploy.yml` (**注意：workflows 目录必须是复数**)

```yaml
name: Deploy Hexo Blog

on:
  push:
    branches:
      - main  # 或者是 master，取决于你的主分支名

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install & Build
        run: |
          npm install --legacy-peer-deps
          npx hexo generate

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

### 3. 开启 GitHub Pages
1.  进入 GitHub 仓库 -> **Settings** -> **Pages**。
2.  **Build and deployment** -> **Source** 选择 **GitHub Actions**。

---

## 五、关键配置：子目录部署

如果你的仓库名**不是** `username.github.io`（例如 `nju-blog`），你的博客地址会是 `https://username.github.io/nju-blog/`。

**必须修改 `_config.yml`，否则页面会白屏或样式丢失：**

```yaml
# 1. 设置完整 URL
url: https://你的用户名.github.io/你的仓库名

# 2. 设置根路径 (非常重要！)
root: /你的仓库名/
```
*   **注意**：配置了 `root` 后，本地预览也需要访问 `http://localhost:4000/你的仓库名/`。

---

## 六、场景 QA 与排坑指南

### Q1: `npm install` 报错 `ERESOLVE overriding peer dependency`
*   **原因**：npm 7+ 对依赖版本检查严格，Hexo 插件生态中存在旧版本依赖。
*   **解决**：始终带上 `--legacy-peer-deps` 参数。
    ```bash
    npm install --legacy-peer-deps
    ```

### Q2: 报错 `Cannot read properties of null` 或莫名其妙的报错
*   **原因**：`node_modules` 损坏或缓存问题。
*   **解决**：执行“核弹级”清理命令：
    ```powershell
    # Windows PowerShell
    Remove-Item -Recurse -Force node_modules
    Remove-Item -Force package-lock.json
    npm cache clean --force
    npm install --legacy-peer-deps
    ```

### Q3: 代码推送了，但 GitHub Actions 没触发
*   **原因**：目录名错误。
*   **检查**：确保配置文件在 `.github/workflows/` 下，而不是 `.github/workflow/`。

### Q4: 部署成功，但打开网页一片空白/样式乱了
*   **原因**：`_config.yml` 中的 `root` 配置错误。
*   **解决**：如果部署在子目录（如 `/blog/`），必须设置 `root: /blog/`。

### Q5: 想要自定义域名
1.  在 `source/` 目录下创建 `CNAME` 文件（无后缀），内容为你的域名（如 `blog.example.com`）。
2.  在 `_config.yml` 中修改 `url` 为你的自定义域名，并将 `root` 改回 `/`。
