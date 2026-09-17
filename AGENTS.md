<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# AGENTS.md

## 项目概览
诗词穿越 RPG 游戏 —— 用户选择诗人 → 进入该诗人的书架 → 选择具体篇目 → 以文字冒险/法庭辩论形式与诗词互动。

- **框架**: Next.js 16 (App Router) + React 19 + TypeScript
- **样式**: CSS Modules (`.module.css`)
- **状态**: XState (gameMachine)
- **测试**: Jest

## 目录结构
```
app/
  page.tsx                 # 首页：诗人选择列表
  poet/[poetId]/page.tsx   # 书架页：博古架陈列该诗人的作品
  play/[dlcId]/page.tsx    # 游戏页：实际游玩
  globals.css              # 全局样式
src/
  components/              # UI 组件
    CurioShelf.tsx         # 博古架书架容器（不规则格子布局）
    CurioBook.tsx          # 线装书组件（竖排/平放/卷轴三种形态）
    curio-shelf.module.css # 博古架 & 线装书样式
    BookCard.tsx           # 旧版书卡（保留备用）
    BookFrame.tsx          # 阅读页框架
    GamePlayer.tsx         # 游戏主控制器
  dlc/                     # DLC 内容加载与编译
    catalog.ts             # 目录数据模型 & 工具函数
    roster.ts              # 诗人名单
    compiler.ts            # DLC 编译器
  game/
    gameMachine.ts         # 游戏状态机（XState）
  ui/                      # 通用 UI 工具
public/
  xuanzhi-bg.webp          # 宣纸纹理背景图（博古架用）
  poets/                   # 诗人头像
```

## 书架系统（博古架风格）

**核心组件**:
- `CurioShelf` — 博古架容器，12 列 CSS Grid 不规则布局，自动分配书籍与装饰格
- `CurioBook` — 单本书组件，支持三种形态：`vertical`（竖排线装）、`horizontal`（平放）、`scroll`（卷轴）

**设计规范**：见 `DESIGN.md`

**配色方案**:
- 主色：靛蓝 `#2c4a6b`（线装书封面）
- 背景：宣纸米黄 `#f0e6d0` + `xuanzhi-bg.webp` 纹理
- 装饰线：暗金 `#b8860b`
- 点缀：印章红 `#a63232`

## 开发命令
```bash
pnpm install     # 安装依赖
pnpm run dev     # 启动开发服务器 (端口 5000)
pnpm run build   # 生产构建
pnpm run typecheck  # TS 类型检查
pnpm test        # 运行 Jest 测试
```

## 测试
- 运行 `pnpm test` 执行所有单元测试（catalog / dlc-schema / eventSchema / gameMachine / compiler / quizBeat）
- 书架页接口：`GET /poet/[poetId]`

## 注意事项
- DLC 内容需要先通过 `pnpm run compile:dlc` 编译到 `generated/dlc/` 目录
- `predev` / `prebuild` 钩子已自动触发编译
- Next.js 16 有破坏性变更，写代码前查阅 `node_modules/next/dist/docs/`
- 故事结束到读词之间可在 `manifest.yaml` 配可选 `easterEgg`；不配则最后一页只有「开始读词」，不会出现「这是什么？」。字段见 `docs/dlc-spec.md`。
- 试评轨迹 YAML 放 `docs/teaching/prompt-lab/cases/`（进 git，quiz-only）。开发环境进总结前会把完整对局（故事+课堂，不含读诗/彩蛋）写到 `assets/sessions/<dlcId>/`。总评 LLM 目前返回「待完成」，作业见 `docs/teaching/README.md`。
- 改 DLC 故事或题目时请升高 `manifest.yaml` 的 `version`。分析本机对局时丢掉 `dlcVersion` 对不上的文件。

## 新增 DLC（带练创作）

当聊到「做某个诗人的某首诗」时，先走带练流程，别直接写 YAML。YAML 字段与图规则见 `docs/dlc-spec.md`。

**中间产物只写到仓库根目录 `assets/<poetId>/<work-slug>/`（诗人 id / 诗词 slug，如 `assets/sushi/shuidiao-getou/`）。** 根目录 `/assets/` 已在 `.gitignore` 中忽略，用来放带练过程文件，避免污染 git。不要写进 `docs/`、`dlc/` 或其他会被跟踪的目录。DLC 包内的 `dlc/.../assets/` 是正式背景/立绘，会进 git，草稿不要放进去。

带练工作单（按环节逐份落盘，确认后再誊进 YAML）：

| 环节 | 中间产物 | 最终落到 |
| --- | --- | --- |
| 信息搜集 | `01_teaching-card.md` 备课卡 | 不直接转 YAML |
| 剧情设计 | `02_story-beats.md` 故事线与关卡 | `content/story.yaml` |
| 读诗教学 | `03_poem-notes.md` 诗词解析 | `content/poem.yaml` |
| 课堂教学 | `04_quiz-design.md` 考题设计 | `content/quiz.yaml` |
| 课堂教学·试评 | 轨迹案例不写 assets，写 `docs/teaching/prompt-lab/cases/<pack-id>/` | 试评用，进 git |
| 生图准备 | `05_storyboard.md` 分镜表 | 生成图进 DLC 包 `assets/` |

开始前：查该目录下已有工作单，据已落内容判断当前进度，接着往下走，不重复劳动。再对照 `dlc/`、`src/dlc/roster.ts`、`generated/dlc/`，确认诗人与篇目是否已登记、命名与目录规范是否一致。

## 平台 MCP：上传课包与拿回使用数据

平台把「提交 DLC」和「拿回使用数据」放在同一台 MCP 上：`https://poem.aibeaver.cn/mcp`。这是**远程 MCP 服务，不在当前项目里、也不在本机** —— 不要去找端口、进程或脚本。远程不用 token，每个工具都靠学员自己的 `userId` 开门。

**工具列表里看不到 `list_roster` / `upsert_poet` / `upsert_work` / `ingest_dlc` / `list_my_dlc` / `usage_manifest` / `download_usage_files` 时**，说明调用方还没配置这个 MCP。把下面这段交给用户，让他加进自己 agent 客户端的 MCP 配置，再重新发起请求：

```json
{ "mcpServers": { "poem-dlc-ingest": { "url": "https://poem.aibeaver.cn/mcp" } } }
```

**userId**：`hh` + 学号，或 `hh_` + 学号（`hh11016863` / `hh_11016863`，大小写不敏感）。学号须是当前在读学员。格式不对或不在名单，工具返回「user id不正确，需要咨询老师」—— 此时停下问老师，不要换别的 id 重试。

每个工具调用都必须带 `userId`。管理台人工上传不走这套校验。

### 1. 上传课包（poem-dlc-ingest）

说明页：https://poem.aibeaver.cn/mcp-how-to

- 只向用户要两样：`userId`、DLC 目录（里面有 `manifest.yaml`）。**不要让用户自己打 zip**，也不要问 poetId / 诗人中文名 / 篇名 —— 从 `manifest.yaml` 读。不要向用户要 token。
- 工具：`list_roster`（先看诗人与篇目）→ 诗人不在名册时 `upsert_poet`（正方形 png/jpg/webp，边长 512–1024px，≤2MB，传 `portraitBase64`）→ `ingest_dlc`（zip ≤30MB，传 `zipBase64`）。
- 打包排除 `.DS_Store`、`.git`、`node_modules`、`__MACOSX`。诗人头像是公共资源，**不要放进 zip**。
- 审核 = 机器校验 + 对照 https://poem.aibeaver.cn/dlc-spec 的评审，可能要几分钟，别中途取消。
- `verdict: accept` → 把返回的 `playUrl` 给用户，结束；`verdict: reject` → 按 `issues[].message` / `fixHint` 改 YAML，**你自己重新打包**再 `ingest_dlc`，不要让用户手动重压。
- 线上课包 id 是 `{manifest.id}-{userId}`：同一 userId 同一 short-id 覆盖自己的包，不会盖到别人的，也不会盖到仓库课包。

### 2. 拿回自己 DLC 的使用数据

说明页：https://poem.aibeaver.cn/mcp-usage

- 只向用户要 `userId`，**不要问要哪个课包** —— 先 `list_my_dlc` 列出来让他挑。
- 工具：`list_my_dlc`（自己已上架、未被隐藏的课包 + 试玩地址）→ `usage_manifest`（使用数据清单：每个文件带相对落盘路径、字节数、更新时间、sha256）→ `download_usage_files`（按清单 `path` 取内容，一次最多 25 个）。
- **只导出「课包归属人 = 本人」的数据**：对局记录按 `sessions`、行为事件流按 `dlcId` 逐条过滤后才输出；请求不属于清单的路径，整单拒绝。
- 默认落到本机 `assets/user_data/`，增量更新：拿清单的 `sha256` 与本机已有文件比对，只下载缺失或变更的文件。目录约定：

  ```text
  assets/user_data/
    manifest.json                      # 上次同步基线
    <dlcId>/sessions/<玩家 slug>-<对局 id>.json
    <dlcId>/events/<玩家 slug>.json
  ```

- 远程 MCP 返回 `contentBase64`，由你写盘：**保持清单里的 `path` 原样**，改文件名会让下次增量重复下载。
- 全程 HTTPS，**没有本地脚本可用**：远程 MCP `https://poem.aibeaver.cn/mcp`，或同源 HTTP —— `GET /api/usage?userId=hh_学号` 取清单，`POST /api/usage`（`{"userId":"…","paths":["…"]}`）取内容。不要去找 `scripts/` 下的工具或 `pnpm` 脚本。

### 禁止

- 伪造或借用别人的 `userId`
- 忽略 blocking 意见反复硬传
- 把诗人头像塞进 DLC zip
- 请求不属于清单的 `path`，或把导出的对局数据再上传成 DLC
