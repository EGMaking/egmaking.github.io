# 存匣 · GitHub 文件托管站

一个纯静态、可直接部署到 GitHub Pages 的文件站：访客页只读，管理页负责上传与删除。

```
index.html     访客页：浏览 / 搜索 / 下载，没有任何写操作
admin.html     管理页：登录后上传、下载、删除
files.json     文件清单（两个页面都读它）
files/         文件实际存放目录
.nojekyll      关闭 Jekyll，按原样发布
```

## 部署

1. 新建仓库，把本目录全部文件推到 `main` 分支。
2. 仓库 Settings → Pages → Build and deployment → Source 选 `Deploy from a branch`，分支选 `main`、目录选 `/ (root)`。
3. 等 1 分钟左右，访问 `https://<用户名>.github.io/<仓库名>/`。

仓库若是用户主页仓库（`<用户名>.github.io`），根路径直接就是站点。

## 两个页面的分工

- `index.html`：只读取 `files.json`，渲染列表、搜索、筛选、下载、复制直链。源码里没有任何上传或删除逻辑。
- `admin.html`：账号密码通过后，用 GitHub Contents API 把文件 commit 到 `files/`，同时更新 `files.json`；删除同理。

## 管理员登录

- 账号 `admin`
- 密码 `123456`

登录只是前端门禁（密码写在 `admin.html` 源码里），只能挡住误操作，不等于真实鉴权。真正的写入权限由 GitHub Token 控制。

## 上传需要配置什么

在 `admin.html` 的「仓库连接」里填：

| 字段 | 说明 |
| --- | --- |
| 所有者 | GitHub 用户名 |
| 仓库名 | 仓库名 |
| 分支 | 一般为 `main` |
| Token | Fine-grained token，需 `Contents: Read and write` |

Token 生成路径：GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens。仓库配置存在 localStorage，Token 只存在当前会话的 sessionStorage，不会写入文件。

## 几点限制

- 上传与删除都会产生 commit，GitHub Pages 需要约 1 分钟重新发布，页面刷新后才看到新文件。
- 仓库若设为 Private，`index.html` 无法公开读取 `files.json`，需要公开仓库（或改为带 Token 访问）。
- 大文件受浏览器与 GitHub API 限制，页面把单文件上限设为 20 MB；GitHub 单文件硬上限为 100 MB。
- 想要真正的登录鉴权与权限隔离，需要后端服务；纯静态站点做不到。
