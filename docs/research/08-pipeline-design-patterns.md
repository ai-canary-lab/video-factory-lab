# AI 视频管线架构级深挖：可复用的设计模式

**调研日期：2026-09-29**

> 本报告是 `07-opensource-video-pipeline-survey.md` 的第三轮架构深挖，聚焦三个学习价值"高"的项目：
> **MoneyPrinterTurbo**（provider 插拔最成熟）、**ArcReel**（先锁资产再生成 + 人工门禁 + 版本回滚 + 费用追踪）、**Toonflow**（无限画布 + Agent + 插件市场 + MCP）。
> hypit 已在 `06-hypit.md` 深挖过，本报告不再重复，仅在对比时引用其 SVML 思想。
>
> **验证口径**：凡标注"直接读过"的，均指 2026-09-29 当天用 browser.open 读过原文（仓库页 / 原始 Markdown / raw 文件）；
> 标注"第三方资料"的，指读过 fork 仓库或第三方分析文档的全文，其函数名、行数级细节未用官方源码逐行复核；
> 凡没读到的，一律写"未找到"，不编造代码细节。

---

## 一、MoneyPrinterTurbo（harry0703/MoneyPrinterTurbo，MIT，约 126.7k stars）

仓库：https://github.com/harry0703/MoneyPrinterTurbo

### 1.1 中间表示（IR）

**没有类似 SVML 的显式 DSL。** 它的"中间表示"是分散的三层，均为直接读过或第三方深读确认：

1. **任务参数对象 `VideoParams`**（pydantic schema）：主题、画幅、音色、字幕样式、BGM、素材源等全部拍平成一个参数对象。WebUI / API / CLI 三种入口最终都收敛到它；批量模式（`cli.py --batch-file ./tasks.json`）的 manifest 就是"CLI 参数做默认值 + 每个对象覆盖 `VideoParams` 字段"的 JSON 数组（第三方 fork README，直接读过：https://github.com/weybercurehub/moneyprinterturbo/blob/HEAD/README-en.md）。
2. **脚本分段**：LLM 生成的文案按段落拆分（`generate_script` → `generate_terms` 提取英文搜索词），段落即后续 TTS/素材/字幕的对齐单位。
3. **SubMaker 词级对齐表**：`app/services/voice.py`（第三方专家文档称约 1500 行，待官方源码复核）中的 `SubMaker` 捕获 TTS 的 `WordBoundary` 事件，得到平行数组 `subs`（词）+ `offset`（100 纳秒单位时间戳）。**这是全管线的时间真相源**，字幕烧录直接消费它（第三方深读：https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/pipeline.md）。
4. **任务级落盘**：`storage/tasks/<task_id>/` 存放各阶段产物；`--stop-at script` 后"script 和 terms 会持久化在任务状态里"，可通过 `GET /api/v1/tasks/<task_id>` 查看（第三方深读：https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/examples.md）。

一句话：它的 IR 是"参数对象 + 词对齐表 + 任务目录"，而不是一份人类可读的剧本 DSL。词级对齐思想与 hypit SVML 同源，但没有"改词自动重排"的编译器链。

### 1.2 模型/渲染后端抽象与路由

**直接读过**官方 README（https://github.com/harry0703/MoneyPrinterTurbo）确认：LLM 侧支持 Kimi/OpenAI/Anthropic/Gemini/DeepSeek/通义千问/Azure/火山方舟/Grok/MiniMax/小米 MiMo，以及 OneAPI/LiteLLM/Ollama/Groq/Pollinations 等统一网关；TTS 侧支持 Edge TTS（免费免 key）、Azure、SiliconFlow、Gemini、MiMo、MiniMax、ElevenLabs、Chatterbox、Kokoro、Fish Audio 等；视频素材侧支持 Pexels/Pixabay/Coverr 库存 + MiniMax H3 / Seedance / WaveSpeed / OFox / MuAPI 等文生视频。

路由机制（第三方深读，未用官方源码逐行复核）：

- **LLM**：`app/services/llm.py`。OpenAI 兼容的 provider 共用 OpenAI SDK + 自定义 `base_url`；Gemini、Qwen/DashScope、Ernie、g4f、Ollama、Pollinations 有**专用分支**；失败重试最多 5 次。配置键按 provider 前缀组织：`llm_provider="openai"` + `openai_api_key` / `openai_base_url` / `openai_model_name`。
- **TTS**：`app/services/voice.py` 的 `tts()` **按 voice_name 前缀路由**：无前缀（`zh-CN-XiaoxiaoNeural` 形式）→ Edge TTS（`azure_tts_v1()`，免费）；`azure-tts-v2:` → Azure Speech SDK；`gemini:` / `siliconflow:` / `mimo:` → 各自函数。非 Edge 引擎不产生词边界，用 `populate_legacy_submaker_with_full_text()` 兜底：按中/英/阿标点切分，按字符数加权分配真实音频时长。
- **新增一个模型要改几处**（第三方资料，待官方源码复核）：OpenAI 兼容网关约 2 处（`llm.py` 分支/配置 + `config.example.toml` 键）；原生协议约 3–4 处（`llm.py` 专用分支、config 段、WebUI 选项、可能的测试）。没有插件式注册表，**新增 provider = 改源码分支**，这是它相对 ArcReel/Toonflow 较弱的一点。

### 1.3 任务编排与状态管理

**核心机制是 `app/services/task.py::start(task_id, params, stop_at="video")` 的阶段门控**（第三方研究文档，直接读过全文：https://github.com/senda-labs/dqiii8/blob/HEAD/docs/research/2026-06-10-moneyprinterturbo.md）：

| 阶段 | 函数 | stop_at 门 |
|---|---|---|
| 脚本 | `llm.generate_script` | `script` |
| 搜索词 | `llm.generate_terms` | `terms` |
| 配音 | `voice.tts` | `audio` |
| 字幕 | `task.generate_subtitle` | `subtitle` |
| 素材 | `material.download_videos` | `materials` |
| 成片 | `video.combine_videos` → `video.generate_video` | `video`（默认） |

任何调用方（WebUI/API/CLI）都可以在任意阶段停下——**这是全仓库最值得抄的单个设计**。状态存储在 `app/services/state.py`，默认内存、可切换 Redis（`enable_redis`，另有 `max_concurrent_tasks` / `max_queued_tasks` 配置）。批量模式：manifest 限 100 个任务 / 1 MiB，全部先校验再执行，单个任务失败不阻塞后续，最后打印一份 JSON 汇总（`total/succeeded/failed/tasks[]`，每项含 `index/task_id/status/result/failed_stage/error`）。**没有发现跨进程断点续跑设计**（状态在内存/Redis，任务目录落盘可事后检查，但没有 resume 语义）——未找到。

### 1.4 灰样/预览/人工确认机制

**未找到显式的灰样或人工门禁设计。** 管线默认全自动一键。最接近的：

- `--stop-at` 阶段门：可在 script/terms/audio 等阶段停下，人工检查 `storage/tasks/<task_id>/` 产物后再继续——是"手动可干预"，不是"强制门禁"。
- WebUI 允许在生成视频前手动编辑文案（第三方教程提及，未独立验证）。
- **付费确认门**（直接读过官方 SKILL.md：https://raw.githubusercontent.com/harry0703/MoneyPrinterTurbo/main/docs/skill/SKILL.md）：`SEEDANCE_CHARGE_CONFIRMATION_REQUIRED` / `WAVESPEED_...` / `OFOX_...` / `METASO_MINIMAX_...` / `MUAPI_CHARGE_CONFIRMATION_REQUIRED` 五个环境变量，Agent 检测到后必须向用户说明"每个 clip 都会计费"，用户明确同意后才加 `--confirm-<provider>-charge` 重跑，**禁止静默添加**。这是"花钱前先确认"的标准实现。

### 1.5 配置与模板系统

**直接读过**官方 README 的"首次配置"节：`config.example.toml` → 复制为 `config.toml`，TOML 分段（第三方研究文档列出段名：`[app] [whisper] [proxy] [azure] [siliconflow] [ui]`，直接读过）。关键键（第三方 skill 文档，直接读过：https://github.com/jeonck/skill-hub/blob/HEAD/skills/moneyprinter-turbo/SKILL.md 与 https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/examples.md）：

```toml
[app]
llm_provider = "deepseek"          # ~22 个可选，见 1.2
deepseek_api_key = "sk-..."
deepseek_model_name = "deepseek-chat"
video_source = "pexels"            # pexels | pixabay | coverr | local | volcengine_seedance ...
pexels_api_keys = ["k1", "k2"]     # 数组，多 key 轮换抗限流
voice_name = "zh-CN-XiaoxiaoNeural" # 前缀即路由（见 1.2）
subtitle_provider = "edge"         # edge（免费，用 TTS 词边界）| whisper
bgm_type = "random"                # random | local | none
enable_redis = false
max_concurrent_tasks = 5
max_queued_tasks = 100

[whisper]
model_size = "large-v3"
```

prompt 模板：支持 `video_script_prompt`（追加指令）与 `custom_system_prompt`（整体替换 system prompt）；fork 的 Pro 版本另有按赛道（finance 等）生成 prompt 的模板管理器（第三方，未独立验证）。**没有版本化的 prompt 模板库**——未找到。

---

## 二、ArcReel（ArcReel/ArcReel，AGPL-3.0，约 5.2k stars）

仓库：https://github.com/ArcReel/ArcReel

> 说明：以下"直接读过"指读过其 fork（cing-max/arcreel）的 `AGENTS.md` 全文（https://github.com/cing-max/arcreel/blob/HEAD/AGENTS.md）——该文件是仓库根目录的开发者契约文档，内容与官方 README 的架构描述一致；另有若干 `agent_runtime_profile/` 下的 skill/profile 文档通过搜索结果读过全文（URL 见各节）。

### 2.1 中间表示（IR）

**最接近 hypit SVML 的项目。** IR 是双层结构，全部落盘为 JSON：

1. **项目级单一真相源 `project.json`**：资产定义（`characters` / `scenes` / `props`，含 description、voice_style、character_sheet 等）+ `episodes` 分集账本（`episode/title/script_file/source_range/hook/outline/ledger_status[planned|consumed|stale]`，顶层 `planning_cursor` 标记下一批规划起点）+ `content_mode`（`drama`/`narration`/`ad`）+ `generation_mode`（`storyboard`/`reference_video`，**创建后不可更改**）+ `schema_version`。**剧本中只引用资产名称，不内联资产定义**（"单一真相源，剧本中仅引用名称"）。
2. **分集剧本骨架 `scripts/episode_N.json`**：骨架种类由 `content_mode × generation_mode` 派生，消费方"一律读 `project.json` 分派，不得从剧本上找该字段"：
   - narration × storyboard → `segments[]`（`novel_text/duration_seconds/segment_break/image_prompt/video_prompt`）
   - drama × storyboard → `scenes[]`（`image_prompt/video_prompt/duration_seconds/utterances/...`）
   - ad × storyboard → `shots[]`（`shot_id=E1S{n}`，`section` 带货框架段落标签，`voiceover_text` 一等口播文案）
   - 任意 × reference_video → 自包含 `video_units[]`（`text/duration_seconds`，正文用 `@[名称]{台词}` 引用语法内联引用资产）
   - `metadata`：`created_at/updated_at/generator`。**条目数与总时长不落盘**，读时计算（"落一份只会与正文漂移"）。
3. **项目目录即 IR 的物理形态**（直接读过 `CLAUDE.narration.md` 全文：https://github.com/arcreel/arcreel/blob/HEAD/agent_runtime_profile/CLAUDE.narration.md）：`project.json`、`source/`（原文）、`scripts/`（剧本 JSON）、`drafts/`（规划中间文件）、`characters/`、`scenes/`、`props/`（资产图）、`storyboards/`、`videos/`、`reference_videos/`、`audio/`、`output/`（成片）。

与 SVML 的异同：同——剧本是结构化数据、可重跑、改上游自动向下传导；异——锚定单位是**秒/分镜**而非词，没有"改词自动重排时间轴"的编译器。

### 2.2 模型/渲染后端抽象与路由

这是三个项目里**能力建模最严谨**的一个（直接读过 fork AGENTS.md 的"供应商能力数据"节）：

- **真相源按字段划分**：视频能力位与各类上限归各 backend 的 `VideoCapabilities`（与请求构造同源，见 `docs/adr/0054`）；图片能力位归 `PROVIDER_REGISTRY` 的 `ModelInfo.capabilities`；其余能力数字与默认 model 归 `PROVIDER_REGISTRY`；自定义供应商（`custom-` 前缀）改读 DB 声明。
- **`supported_durations` 未登记即 fail loud**，无隐性 fallback（见 `docs/adr/0018`）——"用户可选但 backend 不认的档位"被视为 bug 而非静默降级。
- **prompt 模板与智能体运行配置不硬编码具体数值**，占位符由编排层注入。
- **新增一个模型要改几处**：`PROVIDER_REGISTRY` 登记 `ModelInfo`（能力位/价格/默认 model）+ 对应 backend 实现/核对 `VideoCapabilities`（尤其时长、分辨率白名单）。文档明确警告了陷阱：个别 backend（如 vidu 的执行期分辨率白名单）独立于 registry，改 registry 时须同步核对 backend。
- 分层契约由 import-linter 强制：`lib.config < lib.*_backends < lib.custom_provider`（CI 必过）。

### 2.3 任务编排与状态管理

- **GenerationQueue + GenerationWorker**：所有生成任务（分镜/视频/角色/场景/道具/参考视频）统一入队，worker 异步处理，**image / video 两条独立并发通道**；`generation_queue_client.py::enqueue_and_wait()` 封装"入队 + 等待完成"。
- **三态独立记账**（直接读过 generate-video SKILL.md 全文：https://github.com/pattoneirc/arcreel/blob/HEAD/agent_runtime_profile/.claude/skills/generate-video/SKILL.md）：`task_state`（队列任务）/ `provider_checkpoint`（供应商是否已提交）/ `artifact_status`（产物 `current/stale/missing/blocked`）**互相独立、分开陈述**——"任务成功不等于当前产物有效"；`provider_checkpoint.submitted` 为真表示供应商侧很可能已计费。
- **断点续跑**：`mcp__arcreel__generate_video_episode({"script": "episode_1.json", "resume": true})`；`resume_executor.py` 是 worker 的 `_process_resume_task` 入口。stale 产物照常可预览、可导出、可参与成片，**服务端复用、不自动重生**；是否重做由用户明确决定。
- **版本回滚**：历史版本保留在 `versions/`（如 `versions/storyboards`）；"不自动删除、覆盖或重生任何已付费产物与历史版本"。
- **准入语义**：视频批量请求**全有或全无**——准入 `admitted` 时整批入队，`blocked` 或 `confirmation_required` 时一个任务都不入队；明确禁止"把整批拆小先跑通过的一半"（会重复提交已付费 unit）。
- **状态存储**：开发 SQLite（`projects/.arcreel.db`，alembic 迁移），生产 PostgreSQL（asyncpg）；统计字段（scenes_count/status/progress）不存储，由 `StatusCalculator` 读时计算；`lib/data_validator.py` 校验 `project.json` 与剧集 JSON 的结构与引用完整性；实时推送走 SSE（项目事件流），任务中间态由前端轮询 `/api/v1/tasks`。

### 2.4 灰样/预览/人工确认机制

**三项目中最完整**（直接读过 fork AGENTS.md "视觉问题诊断与分镜门禁"节 + generate-video SKILL.md）：

1. **分镜审核门禁**：分镜逐张审核，结果原子合并到 `storyboard_quality_review.items/progress`；**"全部分镜审核通过 → 视频生成"**——任一分镜未审核/过期/失败，视频入口保持关闭（`video_gate`），不得先生成再补审。
2. **定向重画上限**：分镜失败后每轮只重画未通过的分镜并立即复核，**最多三次**；第三次仍失败则关闭视频门禁，等待用户处理。
3. **档位确认**：生成预检把 unit 编排时长投影到供应商申请档位，不一致时返回 `reference_duration_confirmation_required`，逐档位向用户说明后，凭 `confirmed_request_durations` 重发原批次。
4. **reference_video 路线的人工确认点**：只有用户明确认可某个 unit 后才调用 `confirm_reference_video`，不得把确认与生成合并。
5. 导出剪映草稿继续人工编辑（官方 README，直接读过）。

### 2.5 配置与模板系统

- **Agent 运行配置源**：`agent_runtime_profile/`（与开发态 `.claude/` 物理分离），含 `.claude/skills/`、`.claude/agents/`、按 `content_mode` 拆分的 `CLAUDE.*.md` 系统 prompt 变体；`lib/profile_manifest.py` 把配置同步到各用户项目的 `.claude/`，以 manifest + sha256 识别用户改过的文件并保留、不覆盖。
- **prompt 模板**：各 skill 目录（如 `generate-script/SKILL.md`）即流程模板；支持 `--dry-run` **只打印将发送的完整 prompt、不调用 API、不写文件**，用于检查 prompt 质量。
- **资产类型扩展**：`characters.py / scenes.py / props.py` 路由由 `_asset_router_factory.build_asset_router()` 按 `lib/asset_types.ASSET_SPECS` 统一生成，**新增资产类型只需在 spec 注册**——这是"新增一种资产改一处"的范本。
- 用户侧模型配置走 WebUI 设置页（`/settings`）；费用在"生成前后查看费用与实际用量"（官方 README，直接读过）。

---

## 三、Toonflow（HBAI-Ltd/Toonflow-app，MIT，约 16.2k stars）

仓库：https://github.com/HBAI-Ltd/Toonflow-app

> 说明："直接读过"指读过以下原文全文：`docs/development.md`（https://github.com/hbai-ltd/toonflow-app/blob/HEAD/docs/development.md）、`packages/nodeScaffold/readme.md`（https://github.com/hbai-ltd/toonflow-app/blob/HEAD/packages/nodeScaffold/readme.md）、`AGENTS.md`（https://github.com/hbai-ltd/toonflow-app/blob/HEAD/AGENTS.md）、`CONTRIBUTING.md`（https://github.com/hbai-ltd/toonflow-app/blob/HEAD/CONTRIBUTING.md），以及官方 README（仓库页）。

### 3.1 中间表示（IR）

**IR 是"画布 JSON"——节点图，不是线性剧本 DSL。** 直接读过 development.md 与 nodeScaffold 文档确认：

- 工作区内有 `canvases` 列表，每张画布存为一个 JSON 文件；节点是带强类型端口（`dataType: STRING` 等）与 `inputs/outputs/handles` 的组件，边表达数据流。
- 节点通过 `nodeTools.register({name, description, parameters: z.strictObject(...), execute})` 向 Agent 暴露可调用函数；参数 schema 由 Zod 转为 Draft 7 JSON Schema，**同一份 schema 同时服务前端表单校验与 Agent 工具调用**。
- `packages/tools/canvas`（"画布操作"插件）让 Agent 可以 `getCanvas/addCanvas/switchCanvas/addNode/connectNodes/deleteEdge/...`——**Agent 直接改图，图即 IR**。
- 与 SVML 的差异：SVML 是"词锚定的线性事件流"，Toonflow 的 IR 是"节点图 + 画布 JSON"。它的复用单位是节点/插件，不是可重跑的整片 workflow DSL。**画布 JSON 的完整 schema 文件未找到**（只读到节点侧的约定，未找到画布文件格式的 schema 定义）。

### 3.2 模型/渲染后端抽象与路由

**插件四类型之一即"提供方"**（直接读过 development.md"插件与模型扩展"节）：

| 扩展类型 | 用途 | 安装位置 | 格式 |
|---|---|---|---|
| 节点 | 画布交互组件、端口、节点操作 | `data/nodes/` | `.umd.js`（≤20MB） |
| 工具 | Agent 能力（工作区操作/搜索/媒体生成），可带 UI | `data/tools/` | `.tool.js`（≤20MB） |
| 技能 | `SKILL.md` + 附属资料，描述创作流程与约定 | `data/skills/<name>/` | zip/tar（≤20MB） |
| **提供方** | **适配不同平台的模型列表、请求参数和媒体返回结果** | `data/providers/` | **媒体提供方 `.ts`（≤2MB）** |

- 媒体接口定义图片/视频/音频三类扩展点，"实际能力由各提供方实现"；当前内置 **TF-Router** 媒体适配（官方自营中转，源码另行开源 HBAI-Ltd/TF-Router），提供图片与视频生成；也支持本地 ComfyUI 与 LLM。
- 安装走桌面协议 `toonflow://install?type=provider&url=...`，用户确认后下载安装；**提供方不覆盖同名项**（节点/工具/技能同名高版本可更新，提供方不行——显式的不一致，实现时注意）。
- **新增一个模型要改几处**：未找到 provider 编写接口的详细文档（development.md 只给了安装与定位表，`packages/providers/src/` 的内部结构未读到）——**未找到，待验证**。可确认的是新增提供方 = 新增一个 `.ts` 适配文件走市场分发。

### 3.3 任务编排与状态管理

- **编排 = 画布节点执行 + Agent 工具调用**，没有发现 GenerationQueue 式的中央任务队列——未找到（development.md 与 AGENTS.md 均未提及任务队列/状态机；如需确认要读 `apps/server/src` 源码）。
- 服务端是 Bun + Express 5，**文件路由约定**：`apps/server/src/routes/**/*.ts` 由 `core.ts` 扫描，按文件相对路径生成 `/api` 前缀路由，**一个接口一个文件**（`settings/get.ts` → `GET /api/settings/get`），路由注册文件自动生成、禁止手改。
- 状态：配置/插件/项目文件保存在本机 `data/`（Docker 卷 `toonflowData`，gitignore）；全局设置走 `u.conf` 单例；项目列表由前端 Pinia 持久化；工作区文件操作统一走 `useWorkspaceFiles`，路径用工作区内相对路径。
- 桌面端通过 `@toonflow/server/app` 的 `createApp` 复用同一服务；MCP 服务在"设置 → MCP"开启，支持 HTTP 与 stdio，供外部 Coding 工具操作 Toonflow。

### 3.4 灰样/预览/人工确认机制

**未找到显式的门禁/审批机制。** Toonflow 是"人在环"的创作工具——画布本身就是实时预览面。最接近灰样概念的是截图中展示的"3D 导演台与镜头预演"（官方 README 截图列表，直接读过），但其实现与是否可作为评审门禁**待验证**。结论：它的哲学是"人一直在画布上"，而不是"机器跑完等人审批"。

### 3.5 配置与模板系统

- **技能 = Markdown 模板**：`SKILL.md`（frontmatter 含 `name`、`description`）+ 附属资料即"创作流程、方法与操作约定"的模板；技能可全局安装，也可放在工作区 `skill/` 目录，**同名时工作区版本优先**——这是"项目级覆盖全局"的模板优先级范本。
- **节点/工具脚手架**：`packages/nodeScaffold` 与 `packages/toolScaffold` 提供"节点开发脚手架与运行时"，改 `packages/` 源码后 `bun run dev:plugins` 构建并同步到 `data/`；内置插件按构建 hash 同步，第三方插件与用户素材不被覆盖。
- prompt 模板：未找到独立的 prompt 模板库——未找到（流程性 prompt 承载在 skills 的 Markdown 里）。

---

## 四、可直接抄的设计模式（7 条）

> 每条格式：**抄什么**（哪个项目的哪个机制 + 来源）→ **在我们这边怎么落地**（具体到 scripts/ 规划中的文件与数据结构）。
> 我们的 scripts/ 规划（见 `scripts/README.md`，直接读过）：`gen_batch.py`（按分镜表批量生成）、`qc_frames.py`（抽帧质检）、`concat.py`（ffmpeg 拼接）、`subtitle.py`（配音字幕对齐）、`timeline.py`（词锚定时间轴）、`shots_schema.json`（分镜表 Schema）。约定：幂等、可断点续跑、每次运行写 `runs/<timestamp>/` 日志。

### 模式 1：阶段门控 `stop_at` —— 任何阶段都可停下检查再继续

- **抄什么**：MoneyPrinterTurbo `app/services/task.py::start(task_id, params, stop_at="video")`，六个阶段 `script/terms/audio/subtitle/materials/video` 每个都是可停靠点（第三方研究文档，直接读过：https://github.com/senda-labs/dqiii8/blob/HEAD/docs/research/2026-06-10-moneyprinterturbo.md）。
- **落地**：
  - `gen_batch.py` 增加 `--stop-after {script,audio,subtitle,materials,video}`（对应我们的阶段：分镜表解析 → 配音 → 字幕 → 素材/生成 → 拼接）。每个阶段结束把产物写入 `runs/<timestamp>/stages/<stage>/`，并更新 `runs/<timestamp>/manifest.json` 中该 stage 的状态。
  - `timeline.py --dry-run`：只打印将发送给模型的完整 prompt 与预算，不调用 API（抄 ArcReel generate-script 的 `--dry-run`，第三方 skill 文档：https://github.com/binaco/arcreel/blob/HEAD/agent_runtime_profile/.claude/skills/generate-script/SKILL.md）。

### 模式 2：批量清单 + 结构化汇总报告，单任务失败不阻塞整批

- **抄什么**：MoneyPrinterTurbo `cli.py --batch-file ./tasks.json`：manifest 先全量校验再执行、限 100 任务/1 MiB、单个失败继续、最后打印 `total/succeeded/failed` + 每项 `index/task_id/status/result/failed_stage/error`（第三方 fork README，直接读过：https://github.com/weybercurehub/moneyprinterturbo/blob/HEAD/README-en.md）。
- **落地**：
  - `gen_batch.py` 读取批量清单（复用 `shots.yaml`，多项目时读 `batch.yaml`：`[{project, shots_file, overrides}]`），`overrides` 覆盖画幅/音色等单个字段（抄它的"CLI 参数做默认值 + 每项覆盖 VideoParams 字段"）。
  - 输出 `runs/<timestamp>/batch_report.json`：`{total, succeeded, failed, tasks: [{index, shot_id, status, failed_stage, error}]}`。`failed_stage` 精确到阶段名（见模式 1 的阶段划分），方便 `qc_frames.py` 只复检失败项。

### 模式 3：TTS 按 `{provider}:{voice}` 前缀路由 + 统一词级对齐表

- **抄什么**：MoneyPrinterTurbo `voice.py` 的 `tts()` 按 voice 名前缀路由（无前缀→Edge，`azure-tts-v2:`/`gemini:`/`siliconflow:`/`mimo:`→各引擎）+ `SubMaker`（`subs` 词数组 + `offset` 时间戳数组）作为全管线时间真相源；非词级引擎用"按标点切分、按字符数加权分配时长"兜底（第三方专家文档：https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/pipeline.md）。
- **落地**：
  - `subtitle.py` 的 voice 参数格式定为 `{provider}:{voice}`（如 `edge:zh-CN-XiaoxiaoNeural`、`minimax:presenter_female`），路由表写成数据（dict），新增 TTS = 加一行映射 + 一个适配函数，不改主流程。
  - 统一输出 `runs/<timestamp>/word_timings.json`：`[{word, start, end}]`（秒）。`subtitle.py` 烧录字幕与 `timeline.py` 的词锚定都消费这一份表；凡接不到词边界的引擎，用"标点切分 + 字符数加权"生成近似表并标记 `timing_source: "estimated"`。

### 模式 4：花钱前先报价 + 显式确认，确认标志永不静默

- **抄什么**：MoneyPrinterTurbo 的付费门（官方 SKILL.md，直接读过：https://raw.githubusercontent.com/harry0703/MoneyPrinterTurbo/main/docs/skill/SKILL.md）——`*_CHARGE_CONFIRMATION_REQUIRED` + `--confirm-<provider>-charge`，Agent 被明令"禁止静默添加"；ArcReel 的准入三态 `admitted/blocked/confirmation_required`（第三方 skill 文档，直接读过）。
- **落地**：
  - `gen_batch.py` 在调用任何生图/生视频/付费 TTS 之前，先按"镜头数 × 单镜头限价"生成 `runs/<timestamp>/budget.json`（`{items: [{shot_id, model, est_cost}], total_est}`）并打印预算表；**没有 `--confirm-cost`（或交互式输入 yes）就退出**，退出码与 MPT 的 exit code 10 对齐语义（缺确认=停下要输入）。
  - 预算键从各 provider 适配器读取单价（见模式 3 的路由表扩展：每个 provider 声明 `price_per_unit`）。

### 模式 5：资产先行锁定 —— 剧本只引用资产 ID，不内联定义

- **抄什么**：ArcReel `project.json` 单一真相源（资产定义与分集账本）+ "剧本中仅引用名称"；新增资产类型只需在 `lib/asset_types.ASSET_SPECS` 注册一处（fork AGENTS.md，直接读过：https://github.com/cing-max/arcreel/blob/HEAD/AGENTS.md）。
- **落地**：
  - `shots_schema.json` 中每个 shot 的角色/场景/道具字段只允许写资产 ID（如 `character: "lin_xiaoyu"`），不允许内联 `image_prompt` 描述；资产定义收敛到 `assets/assets.yaml`（`{characters: {id: {desc, ref_image, voice}}}, ...`）。
  - `gen_batch.py` 启动时先做**引用完整性校验**（抄 ArcReel `lib/data_validator.py` 的思想）：shots 引用的资产 ID 必须在 `assets.yaml` 存在、ref 图文件存在；缺失则直接报错不入队。校验通过后写 `runs/<timestamp>/assets_locked.json`（资产 ID → 文件 hash），后续阶段只认 hash，资产变更即视为新版本。

### 模式 6：分镜骨架按"内容模式 × 生成路线"派生，骨架失配拒绝入队

- **抄什么**：ArcReel `content_mode`（drama/narration/ad）× `generation_mode`（storyboard/reference_video）二维派生剧本骨架（`segments[]/scenes[]/shots[]/video_units[]`），"消费方一律读 project.json 分派"，"骨架失配时停止入队"（第三方 skill/profile 文档，直接读过：https://github.com/arcreel/arcreel/blob/HEAD/agent_runtime_profile/CLAUDE.narration.md、https://github.com/pattoneirc/arcreel/blob/HEAD/agent_runtime_profile/.claude/skills/generate-video/SKILL.md）。
- **落地**：
  - `shots_schema.json` 定义两种骨架：`shot`（分镜驱动：`image_prompt/video_prompt/duration_seconds`）与 `unit`（自包含单元：`text/duration_seconds`，给"参考成片式"输入用）。`timeline.py` 入口先读骨架声明字段 `skeleton: shot|unit` 做分派，**骨架与字段不匹配直接报错退出**，不在运行时做隐性兼容。
  - 预留 `ad` 式第三模式：我们的营销视频与漫剧共用同一套 schema，用顶层 `content_mode` 区分，避免日后为营销视频另起一套格式（抄 ArcReel ADR-0033 把 ad 做成第三 content_mode 而非另起炉灶）。

### 模式 7：三态独立账本 —— 任务状态 / 供应商提交 / 产物有效性分开记

- **抄什么**：ArcReel `task_state`（队列任务）/ `provider_checkpoint`（供应商是否已提交）/ `artifact_status`（`current/stale/missing/blocked`）三者独立；`provider_checkpoint.submitted=true` = 供应商侧很可能已计费；stale 产物可预览可复用、**不自动重生**、不自动删除已付费产物（fork AGENTS.md + generate-video SKILL.md，直接读过）。
- **落地**：
  - `gen_batch.py` 为每个 shot 维护 `runs/<timestamp>/ledger.json`，每条记录三个独立字段：`task_status`（`queued/running/done/failed`）、`provider_submitted`（bool，**计费分界线**）、`artifact_status`（`current/stale/missing/blocked`）。
  - 语义照抄：改了 prompt 只把旧产物标 `stale`（`concat.py` 默认复用 `current`，`stale` 需显式 `--use-stale` 才参与），**只有 `--regen <shot_id>` 才真正重调模型**；`provider_submitted=true` 的记录计入 `budget.json` 的实际花费，失败重试前先检查该标志，避免重复计费。
  - 断点续跑：`gen_batch.py --resume runs/<timestamp>/` 读取 ledger，只跑 `task_status != done` 的 shot（抄 ArcReel `resume=true` 语义）。

### 模式 8：灰样门禁 —— 贵生成之前必须有一道零成本评审点

- **抄什么**：ArcReel "全部分镜审核通过 → 视频生成"（`video_gate`，分镜失败重画最多三次，第三次失败关闭视频入口等待用户；fork AGENTS.md，直接读过）+ hypit "灰样先行、确认再花钱"（`06-hypit.md`）。
- **落地**：
  - `qc_frames.py` 在抽帧质检后输出 `runs/<timestamp>/qc_report.json`（`{pass_rate, failures: [{shot_id, reason}]}`）+ 人可读的 `review.md`（拼接触帧图）。
  - `gen_batch.py` 进入生视频阶段前检查门禁文件 `runs/<timestamp>/gates/storyboard_approved.json` 是否存在且 `approved=true`；不存在则打印待办清单（哪几个 shot 没过 qc、 ref 图缺哪几张）并退出。批准动作可以是人工写文件，也可以是 `qc_frames.py --auto-approve --min-pass-rate 0.9`。

---

## 五、已验证信息 vs 待验证/传闻

### ✅ 已验证（2026-09-29 直接读过原文）

- 三个仓库页的根目录树、stars、license：MoneyPrinterTurbo（MIT，126,733 stars，`app/`/`webui/`/`cli.py`/`config.example.toml`）https://github.com/harry0703/MoneyPrinterTurbo ；ArcReel（AGPL-3.0，5,225 stars，`server/`/`lib/`/`frontend/`/`agent_runtime_profile/`/`docs/adr/`）https://github.com/ArcReel/ArcReel ；Toonflow（MIT，16,178 stars，`apps/`/`packages/`）https://github.com/HBAI-Ltd/Toonflow-app 。
- MoneyPrinterTurbo 官方 SKILL.md 全文（raw）：`MPT_RESULT` 四件套（`VIDEO_FILE/TASK_DIR/LOG_FILE/RESULT_FILE`）、`latest-result.json` 契约、`~/MoneyPrinterTurbo/.agent-logs/`、`storage/tasks/`、五个 `*_CHARGE_CONFIRMATION_REQUIRED` 付费确认变量与 `--confirm-*-charge` 标志、"禁止静默添加"的明令。https://raw.githubusercontent.com/harry0703/MoneyPrinterTurbo/main/docs/skill/SKILL.md
- MoneyPrinterTurbo 官方 README 的 provider 清单（LLM/TTS/素材三节）与 `config.example.toml → config.toml` 配置流程。https://github.com/harry0703/MoneyPrinterTurbo
- ArcReel fork `AGENTS.md` 全文：三层架构、`ASSET_SPECS` 注册、`GenerationQueue/GenerationWorker`（image/video 双通道）、`enqueue_and_wait()`、project.json 单一真相源、SSE 端点、轮询 `/api/v1/tasks`、`provider_checkpoint.submitted` 计费语义、stale 产物策略、分镜门禁与三次重画上限、`versions/` 回滚、`resume_executor.py`。https://github.com/cing-max/arcreel/blob/HEAD/AGENTS.md
- ArcReel `agent_runtime_profile` 文档全文：`CLAUDE.narration.md`（项目目录树、project.json 核心字段、分集账本 `ledger_status`）https://github.com/arcreel/arcreel/blob/HEAD/agent_runtime_profile/CLAUDE.narration.md ；`generate-script/SKILL.md`（四种骨架的 JSON 结构、`--dry-run`）https://github.com/binaco/arcreel/blob/HEAD/agent_runtime_profile/.claude/skills/generate-script/SKILL.md ；`generate-video/SKILL.md`（全有或全无准入、三态独立、`resume:true`、MCP 工具表）https://github.com/pattoneirc/arcreel/blob/HEAD/agent_runtime_profile/.claude/skills/generate-video/SKILL.md ；ADR-0033（ad 第三 content_mode、`video_units[]` 自包含）https://github.com/arcreel/arcreel/blob/HEAD/docs/adr/0033-ad-third-content-mode-unified-skeleton.md 。
- ArcReel 官方 README 的 pipeline 流程图与"关键阶段可确认、单个素材可重做、历史版本可回滚、生成前后费用追踪、剪映草稿导出"。https://github.com/ArcReel/ArcReel
- Toonflow `docs/development.md` 全文：插件四类型与安装位置（`data/nodes|tools|skills|providers/`）、`toonflow://install` 协议、提供方"不覆盖同名项"、媒体接口三类、内置 TF-Router、MCP 开启方式、monorepo 结构（`apps/{web,server,desktop,updateServer}` + `packages/{nodes,nodeScaffold,tools,toolScaffold,providers,skills,mcp,ffmpeg,startup}`）、构建 hash 同步策略。https://github.com/hbai-ltd/toonflow-app/blob/HEAD/docs/development.md
- Toonflow `packages/nodeScaffold/readme.md` 全文：`nodeTools.register` 机制、Zod→JSON Schema 双用、画布操作插件的 Agent 接口（`getCanvas/addNode/connectNodes` 等）。https://github.com/hbai-ltd/toonflow-app/blob/HEAD/packages/nodeScaffold/readme.md
- Toonflow `AGENTS.md`（文件路由约定：一个接口一个文件，`core.ts` 扫描生成 `/api` 路由）https://github.com/hbai-ltd/toonflow-app/blob/HEAD/AGENTS.md ；`CONTRIBUTING.md`（各包入口表、工作区边界）https://github.com/hbai-ltd/toonflow-app/blob/HEAD/CONTRIBUTING.md 。
- 我们仓库 `scripts/README.md` 的脚本规划（直接读过工作区文件）。

### ⚠️ 第三方资料（读过全文，但非官方仓库，函数名/行数级细节待官方源码复核）

- MoneyPrinterTurbo 架构深读（`task.py::start(stop_at=...)` 阶段表、`state.py` 内存/Redis、`config.example.toml` 段名与 ~22 个 LLM provider 名单、前缀路由表、`SubMaker` 词边界机制、5 次重试）：https://github.com/senda-labs/dqiii8/blob/HEAD/docs/research/2026-06-10-moneyprinterturbo.md 与 https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/pipeline.md
- MoneyPrinterTurbo 批量 manifest 语义（100 任务/1 MiB 上限、`failed_stage` 汇总）：fork README https://github.com/weybercurehub/moneyprinterturbo/blob/HEAD/README-en.md
- MoneyPrinterTurbo 配置实例与"花钱项门控清单"：https://github.com/thalesandrades/modoturbo-br/blob/HEAD/.claude/skills/moneyprinterturbo-expert/examples.md 与 https://github.com/jeonck/skill-hub/blob/HEAD/skills/moneyprinter-turbo/SKILL.md
- ArcReel 的 fork AGENTS.md 内容：fork 与上游同构已用官方 README 交叉验证，但不排除 fork 私有改动混入。

### ❌ 未找到（明确标注，不编造）

- MoneyPrinterTurbo：官方源码级的 `voice.py`/`llm.py` 实现（只读到第三方描述）；跨进程断点续跑机制；版本化 prompt 模板库。
- ArcReel：`GenerationQueue` 的持久化语义细节（队列本身落盘还是仅内存）；费用追踪的具体数据表/账本格式（只确认了"生成前后查看费用与实际用量"的产品语义与 `provider_checkpoint` 的计费分界语义）。
- Toonflow：`packages/providers/` 的 provider 编写接口（新增模型到底改几处）；画布 JSON 的完整 schema；中央任务队列/状态机；显式人审门禁；3D 导演台预演是否可作评审门。
- 三者均未找到 word-level 锚定编译器（hypit SVML 式"改词自动重排"）——该思想仍是 hypit 独有。

---

## 六、对我们流水线设计的直接建议（5 条）

1. **中间表示定为"双层 JSON"：`assets.yaml`（项目真相源）+ `shots.yaml`（分镜骨架）+ `timeline.py` 产出词锚定时间轴。** 抄 ArcReel 的"剧本只引用资产 ID"（模式 5）与 hypit 的"事件锚定在词上"（`06-hypit.md`）：`shots.yaml` 的每句台词带 `words[]`（词/起止），`timeline.py` 负责把"改词"重排为新的时间轴。这是我们区别于 MoneyPrinterTurbo（无显式 IR）和 Toonflow（图 IR）的差异点。
2. **Provider 层先做两类、接口从小做起。** 抄 MoneyPrinterTurbo 的"前缀路由 + 统一词级对齐表"（模式 3）：第一批只抽象 TTS（`subtitle.py` 内）和 LLM（`gen_batch.py` 内），`{provider}:{name}` 前缀路由 + 每个 provider 声明 `price_per_unit`（给模式 4 的预算用）。生视频 provider 先只做"OpenAI 兼容网关"一个适配器——MPT 证明"共用 OpenAI SDK + base_url"能覆盖大多数网关，原生协议分支等真需要再加。
3. **编排层现阶段不要引入 Celery/Redis/GenerationQueue。** 我们的规模用"阶段门控（模式 1）+ 批量清单（模式 2）+ 三态账本（模式 7）+ `--resume`"就够了：全部状态就是 `runs/<timestamp>/` 下的 `manifest.json/ledger.json/budget.json` 三个文件，人可读、可 git、可审计。等日产量真到需要并发 worker 时，再抄 ArcReel 的双通道 GenerationWorker。
4. **把"成本门禁"和"质量门禁"做成硬门，而不是文档建议。** 模式 4（`budget.json` + `--confirm-cost`）与模式 8（`gates/storyboard_approved.json`）应该是 `gen_batch.py` 的默认行为：缺预算确认不调用付费模型，缺灰样批准不进生视频阶段。这是 ArcReel 和 MPT 各自用一半、我们应该合起来的一道墙。
5. **模板与技能用"Markdown + frontmatter"的轻量方案，暂不建插件系统。** 抄 Toonflow 的 skill 约定（`SKILL.md` + `name/description` frontmatter、工作区版本优先覆盖全局）：`prompts/templates/` 下的每个模板就是一个带 frontmatter 的 Markdown（含适用 `content_mode`、变量表、 few-shot 例），`shots_schema.json` 校验变量完备性。UMD 式插件市场是 Toonflow 为生态做的重器，我们当前不需要。

---

*报告人：Muse（video-factory-lab 第三轮专题调研 · 专题B）*
*调研方法：browser.search / browser.open 读取三仓库的仓库页、官方 README、SKILL.md raw 文件、Toonflow 开发文档与脚手架文档、ArcReel fork AGENTS.md 及 agent_runtime_profile 文档、第三方架构分析；所有"未找到"均如实标注，未编造代码细节。*
*License 提示：MoneyPrinterTurbo 与 Toonflow 为 MIT，可直接复用代码；ArcReel 为 AGPL-3.0，本报告只吸收其设计思想，不建议直接复用其代码到闭源商业场景。*
