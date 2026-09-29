# Agent Skill 驱动的视频生产方案专项调研（专题C）

> 调研日期：2026-09-29
> 调研员：video-factory-lab 第三轮调研 · 专题C
> 范围：以 Claude Code / Agent Skill 规范驱动的 AI 视频（短剧/漫剧/营销视频）生产方案，中英文兼查，优先 2026 年信息。

---

## 一、项目清单

| # | 项目 | Repo | Star 量级 | License | 一句话定位 |
|---|------|------|-----------|---------|-----------|
| 1 | drama-skills | https://github.com/zenstory-ai/drama-skills | 约 2.3k（仓库页元数据；其文档自称约 1.8k） | MIT | 11 个 Agent Skill 覆盖短剧/漫剧全流程：原著分析→剧本→视觉设定→分镜→图片/视频提示词→确认后生产→剪辑→审查；文本优先，每集只维护 5 份 Markdown |
| 2 | wind-comic | https://github.com/ChrisChen667788/wind-comic | 约 586 | MIT | 一句话进、整片短剧出的多 Agent 应用（Next.js）：Writer→Director→Style Bible→角色设计→分镜（Vision 审计）→多引擎视频竞速→TTS/口型同步→剪辑师；仓库内含 `.agents/skills` 与 `skills-lock.json` |
| 3 | ai-mandrama-skills | https://github.com/cyuanxv/ai-mandrama-skills | 约 24 | MIT | 中文漫剧端到端 Claude Code Skill 包（Dreamina CLI + edge-tts + ffmpeg）：一句话剧本→14 镜头→配音→字幕/BGM/SFX→横竖双版成片，内置 10 项踩坑速查 |
| 4 | short-drama-skills（LuxReal） | https://github.com/YvonneMovingon/short-drama-skills | 约 88 | MIT | 经 LuxReal 平台工业化验证的 7 个短剧 prompt 技能（叙事拆解/情绪刻画/动作描述/连贯拆分/提示词润色三件套）；`prompt.md` 可复制进任意 LLM，非 Agent Skill 规范 |
| 5 | muki-ai-drama-skills | https://github.com/huang20cheng-blip/muki-ai-drama-skills | 4 | 无开源 License（README 内 Proprietary Use Notice：仅限个人学习/评估/非商业内部使用） | 9 个 Codex Skills：剧本审校与改编、人物/场景提取、角色/场景提示词、Seedance 分镜、海报封面；强调"视觉体系隔离"（国产 3D/国内真人/海外真人/古风仙侠不混用） |
| 6 | brag（/brag、/brag-slim） | https://github.com/latent-spaces/brag | 约 11k（仓库页元数据；2026-09-20 博客称约 5,954，增长极快） | MIT | 一条命令把做好的项目变成 20 秒发布视频：Skill 只写创意简报，渲染交给 HeyGen 开源的 HyperFrames；同一 skill 目录用符号链接同时暴露给 Claude Code / opencode / Codex / Antigravity |
| 7 | HyperFrames（HeyGen） | https://github.com/heygen-com/hyperframes | 未查证 | 未查证 | 开源 HTML-to-video 框架，`CLAUDE.md` 即技能目录：/product-launch-video、/faceless-explainer、/pr-to-video 等 7+ 视频技能，Skill 写简报、代码渲染成片 |
| 8 | claude-remotion-skill | https://github.com/haidrrrry/claude-remotion-skill | 138 | MIT | 教 Claude 用 Remotion（React+TS）做动态设计视频的 Skill：10 条硬规则 + 强制"渲染→抽帧检查→修复→重渲染"闭环 |
| 9 | remotion-dev/skills（官方） | https://github.com/remotion-dev/skills | 约 4.7k | 未查证 | Remotion 官方 Agent Skills：/remotion-create、/remotion-render、/remotion-captions 等 12 个技能，代码化视频生产 |
| 10 | opensquilla meta-short-drama | https://github.com/lzhhhhc/opensquilla-mobile（skill 文件见下） | 未查证（项目内置 skill） | Apache-2.0（SKILL.md 内 provenance 字段自述） | 元技能（kind: meta）：主题→推断风格/角色/镜头数→起草分镜脚本→**暂停等人审**→双参考图锚定→逐镜视频→片头片尾+烧录字幕→MP4；用工作流 DSL 把"等人确认"写成带超时和取消词的 gate 步骤 |
| 11 | arcads-claude-code（经由 aradotso/claude-code-skills） | Skill 包：https://github.com/aradotso/claude-code-skills ；底层 https://github.com/krusemediallc/arcads-claude-code | 未查证 | 未查证 | 营销视频 Agent Skill 包：Seedance 2.0/Sora 2/Veo 3.1/Kling 等模型编排，含"成本门禁（cost gates）"、任务轮询、MASTER_CONTEXT.md 工作区文件 |
| 12 | AiToEarn generating-drama-recaps | https://github.com/zhaikong/AiToEarn（skill 路径：`project/aitoearn-backend/apps/aitoearn-ai/src/core/agent/skills/generating-drama-recaps/SKILL.md`） | 未查证 | 未查证 | 后端项目内嵌 Skill：短剧解说/二创视频生成（AI 写解说稿 + TTS 配音 + 去原字幕）；属业务系统内 Skill，非独立工作流，仅作形态参考 |

**传闻/待验证区**：wind-comic 仓库根目录有 `.agents/skills/`、`skills/`、`skills-lock.json` 三个 skill 相关目录（仓库页文件列表，2026-09 爬取），推测近期引入了 skill 化能力，但其内容与用途**未深挖**，列为待验证，不展开。

---

## 二、它们如何把 SOP 编码成 Agent 可执行的 Skill

### 2.0 先说共识：Agent Skill 的标准文件结构

Agent Skill 规范（Claude Code / Codex 通用）的事实标准结构，来自各仓库的共同实践：

```
skills/<skill-name>/
├── SKILL.md            # 入口：frontmatter（name / description / allowed-tools 等）+ 正文指令
├── references/         # 按需加载的深度知识（drama-skills 每个 skill 下都有 references/）
├── scripts/ 或 tools/  # 确定性脚本（检查器、脚手架），agent 可调用
└── examples/           # 真实产出样例
```

**触发机制只有两种**（已验证，跨项目一致）：

1. **Slash 命令显式调用**：Claude Code 用 `/short-drama`，Codex 用 `$short-drama`（drama-skills README）；Boriwatopal 等同理。
2. **自然语言自动触发**：靠 `SKILL.md` frontmatter 里 `description` 字段的措辞匹配用户意图——description 写得越像"使用场景说明书"，触发越准。ai-mandrama-skills README 原话："跟 Claude Code 聊'我要做 AI 短剧'…它会自动调起对应 Skill"。

**上下文组织**的核心手法是"**分层加载**"：SKILL.md 只放路由和铁律，细节沉到 `references/*.md` 按需读取，避免一次塞爆上下文（drama-skills 每个 skill 都有独立 references 目录；ai-mandrama-skills 把 176 行 master 脚本和完整 prompt 模板沉到 `reference.md`）。

### 2.1 drama-skills：把"制片厂 SOP"写成 11 个技能 + 确定性检查脚本

文件结构（仓库 `skills/` 目录，已验证）：

```
skills/short-drama/                  # 路由 skill：初始化、路由、Look Development、Dashboard
skills/short-drama-novel-analyze/     # 原著抽样快评
skills/short-drama-develop/          # 故事开发、分集地图、导演阐述
skills/short-drama-write/            # 分集剧本
skills/short-drama-assets/           # 人物/场景/道具 + 连续性锁
skills/short-drama-image-prompts/    # 参考图提示词
skills/short-drama-storyboard/       # 分镜 + 冻结关键帧
skills/short-drama-video-prompts/    # 视频提示词
skills/short-drama-produce/          # 确认后生产（外部 adapter）
skills/short-drama-edit/             # 剪辑单 + 成片渲染
skills/short-drama-review/           # 独立审查
每个 skill 下：SKILL.md + references/*.md + suite-ref.json
tools/ 下：creator_markdown_check.py、project_tool.py、edit_tool.py（确定性脚本）
```

编码 SOP 的四个关键设计（均有原文为证）：

1. **五份 Markdown 就是创作事实**（`docs/open-source-short-drama-pipeline.md`）："At most five Markdown files per episode. They are the creative facts; there is no separate database." 每集只维护 `剧本.md`、`视觉设定.md`、`分镜.md`、`图片提示词.md`、`视频提示词.md`（剪辑时加 `剪辑单.md`）。改哪份文件就是改哪一层决定——**状态持久化 = 文件本身**，无数据库。
2. **连续性锁（Continuity locks）**：`视觉设定.md` 里需要跨镜保持的造型写成"一条能原样贴进提示词的短语"，如 `LOCK-JIANGCHEN-DRESS《江晨松枝绿常服》· 锁面：pine-green lapel service jacket`（README 真实样例）。分镜里每个镜头的冻结关键帧必须"回填视觉依据"——写清依据了哪些条目。
3. **三态引用**：`IMG-*`（只是提示词）、`PLAN-*`（待用户提供的参考图槽位）、`REF-*`（项目里真实存在且已检查的图片）。只有 `REF-*` 能绑进视频提示词的参考位——**用命名约定做门禁**。
4. **门禁 = 可执行的 Python 检查器**：`tools/creator_markdown_check.py` 核对五份文档之间的引用，报错说的是**原因**而非"校验失败"，例如：
   ```
   ERROR: SHOT-EP001-002: 冻结关键帧提示词写到人物「周薄森」，视觉依据没有覆盖；……
   ERROR: SHOT-EP001-003: 分镜时长 2 秒与视频提示词 3 秒不一致；视频提示词只能原样照抄已接受的镜头时长
   ```
   （README"写漏了会被指出来"一节，真实报错原文）
5. **确认闸门的形式化**：`short-drama-produce` 流程是"展示有边界的任务（含准确数量、内容、参考、参数、输出和 adapter）→ 用户明确确认 → 执行"。README 明确："A preview, a 'continue', or a budget note is never treated as confirmation."（预览、说"继续"、预算备注都不算确认。）另有版本化设计：commit 记录显示其 skill 图是"version-pinned"，且"定量规范是 craft_default 校准基线而非阻塞门禁"——**有些门是刻度尺，不是闸**。

触发与路由：`short-drama` 是总路由 skill（初始化/路由/Dashboard）；11 个技能也可独立调用、独立安装（`ln -s` 单个目录即可）。分发走 `npx skills add zenstory-ai/drama-skills -y -g`（skills.sh 注册表）。

### 2.2 opensquilla meta-short-drama：把"等人确认"写进工作流 DSL

文件：`app/src/main/python/opensquilla/skills/bundled/meta-short-drama/SKILL.md`（783 行，已读前 125 行原文）。

它的 frontmatter 不是简单的 name/description，而是一套**工作流元数据表**：

- `name: meta-short-drama` / `kind: meta` / `meta_priority: 75`
- `description`（中英双语）：完整描述"何时用、做什么、做到哪停、不做什么"——description 本身就是触发器 + 边界声明
- `request_template`：声明输入字段（story_topic 必填；render_style / character_identity / shot_count 默认 5 等选填；audience、language 有默认值）
- `output_contract`：要求产出必须包含的章节（Story/script summary、Review or adjustment status、Generated media status、Saved deliverable locations）
- `policy_tags: generated-media-review, user-approval-before-media`
- `triggers`：中英关键词列表（"生成短剧"、"make a short drama"、"shot list to final mp4" 等）
- `metadata`：声明依赖（bins: ffmpeg ffprobe）、风险等级（risk: high）、能力清单、可组合的子技能（composition_skills: ai-video-script、short-drama-delivery-audit、subtitle-burner 等）
- `composition`：**步骤编排**，每步有 kind：`llm_chat`（需求提取）→ `agent`（写脚本）→ `tool_call`（write_file 落盘）→ **`user_input`（review_gate：审查门禁）** → `skill_exec`（审后处理）…

最值得抄的是它的 **review_gate** 设计（原文）：
- 以表单形式呈现脚本预览 + "我对风格/角色/分镜数做的假设（标 AUTO_FILLED: yes 的项是我替你填的，你可以改）"
- 用户回复原样存入 catch-all 字段，"不要总结、不要重写"
- **"Empty replies remain invalid and never imply consent."**（空回复永远不算同意）
- 修改只触发免费重拟稿，不代表授权调用媒体商；修改后必须**再次明确说"继续生成"**
- gate 带 `cancel_keywords`（取消/算了/stop/abort）、`timeout_hours: 24`
- 预览里直接给出"本次待授权脚本的美元成本区间"，授权只适用于当前脚本的镜头数和计费时长
- 假设标记机制：`AUTO_FILLED_RENDER_STYLE: yes/no` —— agent 替用户填的默认值必须显式标出来，允许用户逐项改

状态组织：按次运行目录 `meta_short_drama/<meta_run_id>/script.txt`，用户可直接改文件、"下一步会重新读盘"，手动编辑被自然带入流程。

### 2.3 ai-mandrama-skills：SOP 九阶段 + 踩坑库的"经验封装型" skill

文件：`skills/ai-mandrama-pipeline/SKILL.md`（主流程指令）+ `skills/ai-mandrama-pipeline/reference.md`（完整 prompt 模板 + 176 行 master 脚本 + 后台 polling）+ `skills/edge-tts-chinese-roleplay/SKILL.md`（配音专用 skill）+ `examples/case-study-末世招队员第一集.md`（5 小时实战复盘）。

编码方式：SKILL.md 内直接写 **SOP 9 阶段**（剧本一卡→分镜表→角色介绍→资产库三视图→单镜头生图→单镜头生视频→首尾帧→剪辑→成片）+ **10 项关键踩坑速查**（如"服务端单用户视频并发=1，必须后台 polling + watchdog"、"drawtext 中文字幕用 textfile= 避转义"）。本质是把一次真实生产（案例：14 镜头、1m02s、5 小时、credit 耗 400）的全部经验沉淀成"命令模板 + 踩坑修复"，agent 按阶段给出"当前阶段命令模板 + 下一步预告 + 预算估算"。

### 2.4 short-drama-skills（LuxReal）：非规范 skill，"prompt 规则库"

结构：`skills/01-通用叙事拆解/` 下含 `prompt.md` + `prompt.zh-CN.md` + `skill.md` + `example/`。**没有** SKILL.md frontmatter，不走 agent 自动触发；用法是"打开 prompt.md → 复制 → 粘贴进任意 LLM → 接剧本"。定位是"生产验证过的 prompt 规则"而非"agent 可执行的工作流"——说明"skill"一词在生态里有两种含义，调研时需区分。

### 2.5 muki-ai-drama-skills：面向 Codex 的"显式调用 + 边界声明"风格

结构：`.agents/skills/<skill-name>/`（仓库级安装即复制整个 `.agents/` 文件夹到目标仓库根目录）。README 设"Explicit Invocation"一节：按名调用（`$ai-drama-storyboard`）或自然语言匹配 description 触发，"Read the selected Skill's SKILL.md and required references before execution"。

独特设计是 **Visual Systems** 一节：四种视觉体系（国产 3D 动漫 / 国内真人短剧 / 海外真人短剧 / 古风仙侠 3D）的风格短语、身份、资产**不得跨体系混用**，除非任务明确要求——把"风格一致性"写成一条禁令而非技巧。另有"Boundaries"一节明确 skill 不带任何剧本/资产/聊天数据。

### 2.6 brag / HyperFrames / remotion 系：营销与 MG 视频的 skill 模式

- **brag**（`skills/brag/`、`skills/brag-slim/`）：skill **不渲染**，只写创意简报（角度、基调、展示哪些时刻），渲染交给 HyperFrames。工程亮点：同一 skill 目录用符号链接同时挂到 `.claude/skills`、`.opencode/skills`、`.agents/skills`、`.claude-plugin` 四个 agent 发现路径，一次安装全端可用；另走 `npx skills add`（skills.sh）分发。附 `PRODUCT.md`（品牌/用户/成功标准）。
- **hyperframes**（heygen-com/hyperframes，`CLAUDE.md`）：CLAUDE.md 即技能目录，按意图分流（/product-launch-video、/faceless-explainer、/pr-to-video、/embedded-captions、/talking-head-recut、/motion-graphics、/music-to-video）。
- **claude-remotion-skill**（`remotion-motion-graphics/SKILL.md`）：10 条"不可协商规则"（如禁用线性插补、入场必须 2–3 属性联动、所有静帧加 Ken Burns）+ 强制"渲染→抽帧**看**→修复→重渲染"闭环，"Never deliver an unverified render"。
- **kitfunso/claude-config**（`skills/product-launch-video/SKILL.md`，第三方个人配置仓，stars 未查证）："You are the orchestrator. Run each step, verify its gate, and only then continue." 步骤 0/3/6 为用户门禁（user-gated），工作目录固定 `videos/<project>/`，门禁类型定义在 `../hyperframes-core/references/brief-contract.md`——**门禁语义本身也被文件化**。
- **arcads**（经由 aradotso/claude-code-skills 的 SKILL.md）：营销视频 skill 含 **cost gates（成本门禁）**、任务轮询、API 编排；setup 脚本生成 `MASTER_CONTEXT.md` 工作区文件（项目级上下文持久化）。

### 2.7 包装器模式：jeo-skills 的 drama-skills 封装

akillness/jeo-skills 用一个元 SKILL.md（`.agent-skills/drama-skills/SKILL.md`）包装上游 drama-skills（已读全文），手法值得注意：

- frontmatter 增补 `allowed-tools`、`compatibility`（"上游套件需 Python 3.9+，主要中文；生产走 Seedance/GPT Image 2/MiniMax Music，花真钱，必须留在精确预览+明确确认之后"）、`metadata`（tags/platforms/version/source）
- description 里写 "Triggers on:" 关键词列表 + 明确**不接**的相邻任务分流到其他 skill（generic 视频走 `video-production`，条漫走 `webtoon-harness`）
- 正文是**操作模式表**：`install-maintain` / `route-create` / `project-ops` / `produce` / `troubleshoot`，"Choose the smallest mode that answers the request"（选能回答请求的最小模式），并规定"Do not blend `produce` into a normal writing request"（不把生产混进写作请求）
- 配 `scripts/drama-skills.sh doctor/routes` 只读诊断脚本和 `references/install-and-operations.md`、`references/upstream.md`（版本 pin 记录）

---

## 三、与我们的 AGENT.md + sop/ + gates.md 的异同

> 对比基线：`AGENT.md`（角色/工作循环/铁律）、`sop/00-overview.md`（8 阶段地图 Phase 0–7、人机分工表）、`sop/gates.md`（每阶段 checklist 门禁）。`sop/01/02/03` 未逐字读，对比以总览+门禁为准。

| 维度 | video-factory-lab（我们） | drama-skills | opensquilla meta-short-drama | ai-mandrama-skills | wind-comic |
|---|---|---|---|---|---|
| **定位** | 面向 agent 的视频创作知识库（漫剧+营销双产品线）；agent 在**用户项目**里全权执行 | 面向 agent 的短剧/漫剧 skill **工具包**；装进用户已有的 agent 环境 | Agent 平台内置的**元技能**（更大 harness 的一部分） | 个人实战经验沉淀的 skill 包（Dreamina+edge-tts 特定栈） | 独立 **Web 应用**（Next.js），多 agent 在应用内编排，非 skill 规范 |
| **SOP 载体** | `AGENT.md`（手册）+ `sop/*.md`（分阶段手册）+ `gates.md`（门禁） | 11 个 `SKILL.md`（每阶段一 skill）+ `references/` 深度知识 | 1 个 783 行 SKILL.md，内含 composition 工作流 DSL | 2 个 SKILL.md（流程 skill + 配音 skill），SOP 九阶段写在正文 | `services/hybrid-orchestrator.ts` 代码编排 8 个 agent |
| **触发** | 人读 AGENT.md 后按循环执行（无机器触发） | slash 命令 `/short-drama` 或自然语言（description 匹配） | triggers 关键词表 + kind:meta 路由 | 自然语言（"我要做 AI 短剧"） | 用户在 Web UI 输入一句话 |
| **阶段划分** | Phase 0 立项→1 剧本→2/3 角色场景→4 分镜→5 生成→6 配音→7 后期（8 阶段） | 11 技能 ≈ 11 阶段（多出原著分析、故事开发、剪辑单、审查） | 单 skill 内多步骤（需求提取→脚本→门禁→生成→交付） | 9 阶段（剧本一卡→…→成片） | 8 agent 接力（Writer→Director→Style Bible→…→Editor） |
| **状态持久化** | `runs/<日期>/notes.md` 记录决策过程；产出物按约定路径落盘（gates 要求 `runs/<时间>/` 记录完整） | **五份 Markdown 即事实**，无数据库；文件名即版本；Dashboard 可视化 | 按次运行目录 `meta_short_drama/<run_id>/`，用户可直接改文件续跑 | 未明确（命令模板式，靠 agent 会话） | **双驱动 DB**（SQLite⇄PostgreSQL）+ 资产库；每件 artifact 独立可复用、阶段可单独重跑 |
| **门禁机制** | `gates.md` checklist，agent 人工打勾；"没通过就不进入下一阶段" | `creator_markdown_check.py` **脚本自动核对**文档间引用，报错给原因；另有"确认闸门"（produce 前） | `review_gate` 步骤：表单+取消词+24h 超时；空回复≠同意；二次确认 | 无显式门禁（靠"下一步预告"人工推进） | Vision 审计（分镜<70 分自动重生成）、Cameo 重试（<75 分）、四维度发布门禁、预算护栏 |
| **花钱前的确认** | Phase 4 灰样门禁："灰样已过…用户确认故事节奏成立，再进入烧钱的视频生成" | preview→explicit confirmation→produce；"预览/说继续/预算备注都不算确认" | 预览脚本+**成本区间**→明确说"继续生成"→外部调用；修改只触发免费重拟稿 | 有"预算估算"，但确认语义未形式化 | 按项目成本归因 + budget guard |
| **审查** | Phase 5 三处质检点记录；用户终审 | `short-drama-review` 独立 skill：只写 findings，不改源文件，"Review writes findings; the owner decides" | composition_skills 含 short-drama-delivery-audit | 无 | pacing audit（冲突/反转/悬念）、抛光工作室审计 |
| **一致性手段** | 角色卡/场景卡（含定稿图、seed、voiceId） | 连续性锁（可原样粘贴的短语）+ 冻结关键帧回填视觉依据 + IMG/PLAN/REF 三态 | 双参考锚定（全员身份参考图 + 单镜构图图同时约束） | 三视图锚定 + image2image | Style Bible Frame + 角色 8 维 DNA + cref/sref |
| **可复现性** | 铁律："模型/prompt/seed/参考图版本，没有记录=没发生过" | 供应商凭据不进项目；生产记录结果落盘；中断后 `audit`→`collect --job-id` 取回（不重复计费） | 输出契约要求记录交付物位置 | case-study 记录 credit/时间/踩坑 | artifact 全持久化，阶段可重跑 |
| **用户角色** | "用户永远只做两件事：给创意、做确认" | 创作者做判断（review 的 owner），确认后才生产 | 自由评审（可改、可取消），明确授权 | 用户按模板执行命令 | 在时间线 UI 里直接导演（可逐帧检查、只重拍坏掉的两秒） |
| **分发** | GitHub 仓库（待 push） | `npx skills add`（skills.sh）/ 软链 `~/.claude/skills` / 自然语言安装 | 随 opensquilla 平台内置 | git clone + 软链 | Docker/自托管 Web 应用 |

**核心差异一句话**：我们是"**给 agent 读的手册**"（人/agent 主动遵循），drama-skills 是"**agent 可安装执行的技能包**"（机器可触发、脚本可校验），opensquilla 是"**带表单门禁的工作流程序**"，wind-comic 是"**应用**"。四者在"SOP 编码"这件事上处于不同抽象层，但解决的是同一问题：**让 agent 在长流程里不跑偏、不瞎花钱、状态可恢复**。

---

## 四、可直接借鉴的模式

### 模式 1：Skill 文件包结构（SKILL.md + references/ + tools/ + examples/）
drama-skills、ai-mandrama-skills、jeo-skills 共同验证：SKILL.md 只放"路由+铁律+最小指令"，深度知识下沉到 `references/` 按需加载，确定性工作交给 `tools/*.py`。**我们现在的 `sop/*.md` 是"一整本手册"，可以按 skill 包结构重组**：每个阶段一个目录（`SKILL.md` 概述 + `references/` 细节 + `tools/` 检查脚本），AGENT.md 只保留路由表。这样未来无论人读还是 agent skill 化调用，都能"按需加载、不一次塞爆上下文"。

### 模式 2：门禁脚本化——gates.md 的每一项都应有"机检/人检"标注
drama-skills 的 `creator_markdown_check.py` 证明：文档间引用一致性（镜头时长 vs 提示词时长、关键帧人物 vs 视觉依据）完全可以机检，且报错要给**原因**而非"失败"。我们 `gates.md` 目前全靠 agent 人工打勾，建议把可机检项（文件存在、版本号冻结标记、shots.yaml 字段完整性、runs 记录完整性）写成脚本，`gates.md` 每项标注 `[自动]`/`[人工]`。

### 模式 3：确认闸门的精确语义（"什么算确认"）
三家不约而同地形式化了确认语义：
- drama-skills："预览、说'继续'、预算备注都不算确认"
- opensquilla："空回复永远不算同意；修改只触发免费重拟稿；二次明确说'继续生成'才调用"
- kitfunso 的 SKILL.md：user-gated 步骤显式编号
我们 AGENT.md 写了"用户亲口确认"，但没定义"亲口"的边界。建议把确认语义写死：**确认必须针对"本次预览的批次（内容+数量+参数+成本）"的明确肯定；模糊回复、空回复、话题转移都不算确认，agent 必须追问**。

### 模式 4：阶段状态的显式持久化（断点续做）
- drama-skills：五份 Markdown 即事实，无 DB
- opensquilla：`meta_short_drama/<run_id>/` 运行目录 + 用户可直接改文件续跑
- wind-comic：artifact 全持久化、阶段可独立重跑
- arcads：`MASTER_CONTEXT.md` 工作区文件
我们有 `runs/<日期>/notes.md`，但缺一份**机器可读的阶段状态**（当前在 Phase 几、各阶段冻结版本、待用户确认事项）。建议加 `projects/<项目>/status.yaml`（或 status.md），agent 每完成一阶段更新，断点续做时先读它。

### 模式 5：连续性锁——把"不能变的东西"写成可原样复用的短语
drama-skills 的锁面短语（`pine-green lapel service jacket` 中英双语、原样贴进提示词）+ 冻结关键帧回填视觉依据 + IMG/PLAN/REF 三态，比"角色卡里写一段描述"更抗漂移。我们 Phase 2/3 的角色卡/场景卡可以加一个"连续性锁"字段：每条锁一句话、双语，生成 prompt 时**原样复制**，检查脚本核对"锁面短语是否出现在本镜 prompt 中"。

### 模式 6：成本门禁与预览绑定
arcads 的 cost gates、opensquilla 的"预览里直接给美元成本区间且授权只适用于当前批次"、drama-skills 的"提交即计费、中断后 collect 取回不重投"。我们 Phase 4 已有"灰样先行"，可再加一条：**进入 Phase 5 前，agent 必须给出本批生成的"镜头数×单价=总成本"预览，用户确认的授权只对该批次有效；任务中断后先查单后重投，不静默重跑**。

---

## 五、已验证信息 vs 待验证/传闻

**已验证**（来自仓库页面元数据或文件原文，2026-09 爬取）：
- 各项目 star 数、license、创建时间（见表一；muki 的 4 stars、非开源声明来自其仓库页与 README 原文）
- drama-skills 的 11 技能列表、五份 Markdown 事实、三态引用、检查器报错原文、确认闸门语义（README 与 `docs/open-source-short-drama-pipeline.md` 原文）
- opensquilla SKILL.md 的 frontmatter 字段与 review_gate 机制（文件前 125 行原文，共 783 行）
- ai-mandrama-skills 的 9 阶段 SOP、10 踩坑、reference.md 结构（README 原文）
- brag 的多路径符号链接安装法与 skills.sh 分发（README 原文；star 数 11,079 为仓库页元数据）
- wind-comic 的 8 agent 链路、586 stars、MIT、v12.455（仓库页原文）

**待验证/传闻**：
- wind-comic 仓库内的 `.agents/skills/`、`skills/`、`skills-lock.json` 内容与用途：仅在文件列表中看到，**未深挖**，不确定是其 agent 内部 skill 还是对外 skill 包
- 各项目 star 数为爬虫缓存的页面元数据，实时数字可能浮动；brag 的博客（2026-09-20）称约 5,954，与当前页面的 11,079 差异较大，取页面元数据为准但标注了出处时间差
- hyperframes、remotion-dev/skills、aradotso/claude-code-skills 的 license 与 star：本次未逐一打开仓库页，表中标"未查证"，**未编造**
- drama-skills 各 SKILL.md 正文全文：本次读了 README、docs、DESIGN 摘要与 jeo-skills 包装器全文，未逐字读 11 个 SKILL.md；引用均注明出处文件
- opensquilla SKILL.md 后 658 行（composition 后续步骤、具体 prompt 模板）未读

---

## 六、对我们仓库 AGENT.md / sop/ 的直接改进建议

1. **`sop/gates.md`：每项门禁标注 `[自动]`/`[人工]`，并新增 `scripts/gate_check.py`**
   在每个 Phase 的 checklist 条目后加标注，例如 Phase 4 的"`shots.yaml` 已冻结"标 `[自动]`（脚本校验文件存在+字段完整+版本号），"用户亲口确认剧本定了"标 `[人工]`。新建 `scripts/gate_check.py --phase N --project <项目>`，输出通过/失败及**失败原因**（学 drama-skills 检查器"报错给原因"的风格）。`gates.md` 顶部加一行："先跑脚本，脚本全绿再人工确认剩余项"。

2. **`AGENT.md`："铁律"区新增一条"确认语义"**
   原文建议："用户确认必须针对本次预览的具体批次（内容/数量/参数/成本）作出明确肯定；'继续'、空回复、表情、话题转移都不算确认，未获明确确认不得调用任何花钱的接口；进入烧钱阶段前必须先给出成本预览。"（综合 drama-skills 与 opensquilla 的确认闸门。）同时把"灰样先行"从 gates.md 的 Phase 4 门禁提升为铁律级表述。

3. **新增 `projects/<项目>/status.yaml`：机器可读的阶段状态文件**
   字段：`phase`（当前阶段）、`frozen`（各阶段冻结版本，如 `script: v1.0`）、`pending_user`（待用户确认事项列表）、`last_run`（上次 runs 目录）。`AGENT.md` 工作循环第 1 步改为"先读 `projects/<项目>/status.yaml` 确认当前阶段与断点"。这解决了多会话/长流程断点续做无据可查的问题（对标 drama-skills"文件即事实"与 opensquilla 运行目录）。

4. **`sop/01-preproduction.md`（或角色卡/场景卡模板）：加"连续性锁"字段**
   在角色卡/场景卡模板中新增一节"连续性锁"：每条锁一句话、**中英双语**，要求生成 prompt 时原样复制；`scripts/gate_check.py` 在 Phase 4/5 检查"本镜 prompt 是否包含其角色全部锁面短语"。直接抄 drama-skills 的 `LOCK-<NAME>` 命名与"锁面原样粘贴"机制。

5. **`AGENT.md` 顶部加"路由表"（为 skill 化预留）**
   在"你的角色"之前加一个 10 行以内的表：每行"当用户说…→读…→产出…"（如"当用户给出一句话创意→读 sop/00-overview.md Phase 0→产出 brief.md"），并声明"本仓库不适用场景"（如：用户只想单张生图、只想写小说）。这是 SKILL.md `description` 即触发器的轻量版：今天给人/agent 读，将来拆成 skill 包时每行就是一个 skill 的 description。`INDEX.md` 同步加一行指向该路由表。

---

*报告完。 repo 地址与文件路径引用均来自 2026-09-29 前后的公开页面爬取；未找到的信息已按"未查证/未找到"标注，未编造。*
