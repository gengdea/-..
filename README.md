# 炼金室 

> 杂料进，成稿出。

炼金室是一款 Windows 桌面端「内容转化工作台」：把 PDF、Word、图片、纯文本等零散材料丢进来，经过解析、分析与 AI 生成，直接产出结构化、可修改、可导出的成稿（HTML / PDF / Word / Markdown）。

技术形态是 **Electron + React + Vite** 的本地优先应用：材料与历史默认只存本机，只有在你主动点击分析或生成时，内容才会发送给你自己配置的模型接口。

---

## 1. 核心功能

| 模块 | 说明 |
| --- | --- |
| 三步工作流 | ① 定义任务与材料 → ② 分析确认 → ③ 生成结果，每一步都可回退重来 |
| 材料解析 | PDF（pdfjs-dist）、DOCX（mammoth）、图片 OCR（tesseract.js，中英混合）、TXT/MD/CSV/JSON/HTML/LOG 纯文本；旧版 `.doc` 与音视频暂不支持 |
| 任务定义 | 身份（18 种职业）+ 创作目的（摘要/文案/润色/归纳/问答/数据分析/创意发散/翻译/自定义）+ 补充信息 |
| 生成设置 | 内容程度（稍微精简 / 适度精简 / 尽量详细）、图形表达（结构图 / 问题树 / 思维导图 / 不使用图）、名词处理（保留原词 / 首次解释 / 全部大白话） |
| AI 模型 | 内置 DeepSeek、通义千问、Kimi、MiniMax、Claude 预设与自定义接口；支持思考程度（关闭/低/中/高）与「拉取模型」列表 |
| 图形化正文 | 模型直接输出 `flow / tree / mind / bars / metrics / table / callout / states` 等内容块，图形内嵌在正文对应位置，而非末尾附录 |
| 结果修改 | 生成后可预览全文，并直接用自然语言指出要改的点，由 AI 重写 |
| 导出 | HTML（可双击打开）、PDF（Electron 打印管线）、Word（docx）、Markdown |
| 历史任务 | 本地保存、搜索、按时间（今天/昨天/近 7 天/近 30 天/本月/上月/自定义）与状态筛选、日历视图、重新打开继续编辑 |
| Skill | 6 个内置 Skill（大白话表达、面试追问、事实核验、结构化总结、会议纪要、学习笔记）+ 从公开 GitHub 仓库或本机 `SKILL.md` / 文件夹导入 |
| 模板 | 面试准备资料、会议重点与行动项、学习笔记、一页纸速查、产品需求摘要、大白话改写 |
| 界面 | 浅色/深色主题、自定义标题栏、启动闪屏、IP 形象「小转 CT-01」状态动画 |

> 未配置模型时，应用会走**本地演示结果**（`generationMode: "demo"`），仅用于走通流程，不可作为正式产出。

---

## 2. 技术栈

- **运行时**：Electron 45（`contextIsolation: true`、`sandbox: true`、`nodeIntegration: false`，仅通过 `preload.cjs` 暴露白名单 IPC）
- **前端**：React 18 + TypeScript 5.7 + Vite 6 + lucide-react
- **解析**：pdfjs-dist、mammoth、tesseract.js
- **导出**：docx（Word）、`webContents.printToPDF`（PDF）
- **打包**：electron-builder（NSIS 安装包 + 便携版）

---

## 3. 目录结构

```
炼金室/
├─ electron/
│  ├─ main.cjs              # 主进程：窗口、IPC、密钥加密、Skill 导入、导出
│  ├─ model-api.cjs         # 模型接口：OpenAI 兼容 / Anthropic 协议、模型列表
│  └─ preload.cjs           # 渲染进程可用的 window.desktop 白名单 API
├─ src/
│  ├─ App.tsx               # 主应用：页面、工作流、状态与本地存储
│  ├─ file-parser.ts        # 材料文本提取（PDF/DOCX/图片 OCR/纯文本）
│  ├─ theme.css             # 全站样式与主题变量
│  └─ components/           # TitleBar / BrandLogo / XiaozhuanFigure / SplashScreen
├─ assets/                  # 打包用图标（ASCII 文件名，避免打包器编码问题）
├─ IP+logo/                 # 品牌方案、IP 设定、Logo 源文件与生成提示词
├─ scripts/
│  ├─ make-portable-kit.mjs # 生成「便携版 + 一键创建桌面快捷方式」分发包
│  └─ smoke-check.mjs       # 模型接口模块冒烟测试
├─ tools/
│  ├─ make-shortcut.ps1     # 创建桌面快捷方式（自动定位 exe）
│  └─ desktop-target.txt    # 本机桌面目录覆盖配置（不进仓库）
├─ 一键创建桌面快捷方式.cmd
├─ index.html               # 入口（含 CSP 与主题预加载）
├─ vite.config.ts           # base: "./"，固定端口 5173
└─ package.json             # 脚本 + electron-builder 配置
```

---

## 4. 快速开始

### 环境要求

- Windows 10/11 x64
- Node.js **≥ 22.12**（Electron 45 的 engines 要求）
- npm 10+

### 安装与开发

```bash
npm install
npm run dev      # 启动 Vite(127.0.0.1:5173) + Electron 窗口
```

> `dev` 使用 `concurrently` + `wait-on`，会等 Vite 就绪后再拉起 Electron；关闭窗口即结束本次会话。

### 只跑网页预览

```bash
npm run build:web   # tsc -b && vite build，产物在 dist/
npm run preview
```

浏览器预览下 AI 请求、导出、Skill 导入等桌面能力不可用（`window.desktop` 不存在），会提示改用桌面版。

### 构建安装包

```bash
npm run build
```

产物输出到 `release/`：

| 文件 | 用途 |
| --- | --- |
| `炼金室-0.1.0-setup-x64.exe` | NSIS 安装版，安装后自动创建开始菜单与桌面快捷方式 |
| `炼金室-0.1.0-portable-x64.exe` | 便携版，免安装；首次启动会解压运行环境，耗时略长 |
| `炼金室-0.1.0-win-x64-portable-kit.zip` | 便携版 + 快捷方式脚本 + 使用说明的分发包（由 `npm run pack:kit` 生成） |

### 创建桌面快捷方式

```bash
npm run shortcut
# 或双击 一键创建桌面快捷方式.cmd
```

脚本依次在以下位置找 exe：脚本同目录 → `release/win-unpacked` → `release/` 下的便携版 → `%LOCALAPPDATA%\Programs\炼金室` → `%ProgramFiles%\炼金室`。
桌面目录优先级：`tools/desktop-target.txt` 首行 → 环境变量 `ALCHEMY_DESKTOP_DIR` → Windows 默认桌面。

---

## 5. 脚本一览

| 命令 | 作用 |
| --- | --- |
| `npm run dev` | 开发模式（Vite + Electron 热更新） |
| `npm run build:web` | 类型检查 + 前端构建 |
| `npm run build` | 前端构建 + electron-builder 出安装包与便携版 |
| `npm run pack:kit` | 打包便携版分发 zip（需先 `npm run build`） |
| `npm run shortcut` | 在桌面创建启动快捷方式 |
| `npm run test:smoke` | 校验模型请求体与模型列表解析逻辑 |
| `npm run preview` | 预览 `dist/` 网页产物 |

---

## 6. 配置模型

进入「设置 → AI 模型」，点击右上角「修改」后可编辑：

1. **模型厂商**：DeepSeek / 通义千问 / Kimi / MiniMax / Claude / 自定义
2. **请求地址**：选择预设会自动带出默认 Base URL（Claude 走 Anthropic 协议，其余走 OpenAI 兼容协议）
3. **API Key**：仅保存在本机，优先使用系统安全存储（Electron `safeStorage`）加密
4. **模型**：可从内置列表选择，或点「拉取模型」调用 `/models` 接口获取
5. **思考程度**：关闭 / 低 / 中 / 高

保存后可测试连通性。配置默认只读，防止误改。

---

## 7. 数据与隐私

- 数据目录：`%APPDATA%\content-transform-desktop`（改名为「炼金室」后仍沿用旧目录，避免历史数据丢失）
- 密钥文件：`content-transform-secrets.json`（`safeStorage` 加密后落盘，已在 `.gitignore` 中排除）
- 任务、Skill、设置存于渲染进程 `localStorage`（键前缀 `ct.`），任务内容过大时会提示尽快导出
- 只有你主动触发分析/生成时，材料文本才会发送到你配置的模型接口；应用本身不内置任何上报或统计服务
- 接口地址仅允许 `http/https`，且禁止携带用户名密码；GitHub Skill 导入仅限 `https://github.com` 公开仓库

---

## 8. 常见问题

| 现象 | 排查方向 |
| --- | --- |
| `dev` 启动后窗口空白 | 确认 5173 端口未被占用（`strictPort: true` 会直接失败） |
| 安装包内图标/资源缺失 | 打包只用 `assets/` 下 ASCII 文件名资源，中文文件名会被打包器漏掉 |
| 导入 Skill 失败 | 仓库必须是 github.com 公开仓库，且地址不能带文件路径；本机导入需含 `SKILL.md` |
| 图片 OCR 很慢/无文字 | 首次识别需加载 `chi_sim+eng` 语言包；图片过于模糊会提取不到文字 |
| 导出 PDF 失败 | PDF 走 Electron 打印管线，需使用桌面版并确保导出 HTML 内容可渲染 |
| 音频文件报错 | 当前版本未接入语音转写服务，请先转文本再导入 |

---

## 9. 品牌资源

- IP：**小转 CT-01**，定位「会把零散材料嚼一嚼、再吐出清晰成稿的内容转化机」
- 标准色：番茄橙 `#E8582A`、荧光黄 `#FFD95A`、暖纸白 `#FFFDF8`、墨黑 `#211C17`、珊瑚粉 `#FF9B7A`
- 详细设定见 `IP+logo/品牌说明.md`；Logo 与 IP 源文件在 `IP+logo/` 下
- 应用图标以 `assets/content-transform.ico` 为准（16–256px 多尺寸）

---

## 10. 许可

`UNLICENSED`（私有项目，未授权公开分发）。
