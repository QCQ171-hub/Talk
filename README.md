# Talk · AI 智能体社区

像 QQ 一样的 AI 角色扮演聊天软件。可创建智能体、在广场分享、继续聊天、搜索用户与智能体，
所有发布数据以 GitHub 仓库 [`QCQ171-hub/Talk`](https://github.com/QCQ171-hub/Talk) 为中枢存储。

## 特殊说明

因为这个项目要存储数据，源代码会发上Releases

## 架构
- **后端**：`server/server.js` — 零依赖（仅用 Node 24 内置 `http` / `fetch` / `node:sqlite`）。
  托管前端、代理智谱/OpenAI 兼容聊天（流式）与生图、读写 GitHub 数据仓库、SQLite 本地存储、调用 ffmpeg 做 1:1 头像裁剪、违禁词审核。
- **前端**：`client/` — 原生 **Vite** 工程（无框架），构建产物 `client/dist`。
  侧边栏 SVG 图标悬停展开、广场流动卡片、QQ 风格聊天弹层、自定义主题、创建智能体、我、设置。
- **启动器**：`launcher/launcher.cpp` → `Talk.exe` — 纯 Win32，g++ 静态编译，双击起后端；浏览器由后端自动打开（`--no-open` 可关，dev 模式不自动开）。

## 运行（开发）
```bash
# 后端（serve 模式托管已构建的前端）
node server/server.js                 # 打开 http://127.0.0.1:8899

# 前端热更新开发（可选）
cd client && npm install && npm run dev   # http://127.0.0.1:5173（代理到后端 8899）
npm run build                             # 生成 client/dist
```

## 发布给最终用户（打包目录）
保持如下目录结构后双击 `Talk.exe`：
```
Talk/
  Talk.exe
  server/server.js
  client/dist/...
  data/            # 运行时自动创建：config.json / talk.db / uploads/
```
用户机需装 **Node.js 18+**（后端零依赖，无需 `npm install`）。ffmpeg 在 PATH 时头像走裁剪，否则原样使用。

## 首次使用
1. 打开后进入 **设置**，填入：文字 API（默认预填智谱 `glm-4.5-flash`）、生图 API（默认 `agnes-image-2.5-flash`）、
   GitHub Token（`ghp_` 开头，需对 `QCQ171-hub/Talk` 有写权限）与你的 GitHub 用户名。
2. **创建智能体**：填名字/性别/对外简介/真实人设/开场白，上传头像（自动 1:1 裁剪）或“AI 生成头像”。
   本地保存随便存；点“发布到广场”会做违禁词审核后写入 GitHub 仓库。
3. **广场**：浏览他人发布的智能体，点击直接开聊。**聊天**页保留聊过的智能体可继续。**我** 管理玩家人设。

## GitHub 数据仓库结构
```
index.json                       # 全局注册表：next_num / agents[] / users[]
<15位大写36进制ID>/              # 智能体，从 000000000000001 递增
  data.json                      # 人设 + owner_github + image 名 + comments[]
  image.png|jpg|jpeg             # 头像（json 中 image 字段声明文件名）
<GitHub用户名>_user/             # 玩家主页（绑定 GitHub 后可被搜索/访问）
  data.json                      # 玩家 name/gender/bio/persona + image
  image.png
```

## 安全说明
- 密钥仅存本机 `data/config.json`（用户自填）。测试时通过环境变量注入的密钥**不会**写入发布配置。
- 发布数据走后端代理访问 GitHub（规避浏览器 CORS），Token 不暴露给前端页面。
