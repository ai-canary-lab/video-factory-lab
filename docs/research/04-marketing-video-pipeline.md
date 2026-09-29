# 专题4：AI 产品营销视频流水线 —— 调研报告

> 调研日期：2026-09-29
> 调研员：专题4 子调研员（公开网络资料，中英文来源）
> 说明：本报告所有链接均为调研当日通过搜索抓取到的文本页面；未登录、未实际使用任何工具，结论以"已验证/待验证"区分标注。

---

## 1. 执行摘要

2026 年 AI 产品营销视频的生产范式已经从"单点生成"转向"**连通式生产系统**"：brief → 脚本 → 资产 → 分镜生成 → 剪辑合成 → 变体批量 → 合规标识，各环节共享产品真相（product truth）与品牌上下文。核心趋势有四条：

1. **脚本结构收敛**：短视频营销几乎全部采用 Hook(0-3s) → 问题/痛点 → 产品演示 → 证据/证言 → CTA 的五段式或四段式骨架，分歧只在时长与命名。
2. **UGC 风格成为主流形态**：AI 生成的"真人测评感"短视频（自拍视角、手持运镜、生活化场景）是投流与种草的主力格式，工具链围绕"一人设 + 一产品参照 + 多场景变体"组织。
3. **模型按镜头选用，而非按项目**：2026 年共识是没有单一最强模型——Veo 3.1 叙事与原生音频最强，Kling 3.0 电影感与性价比，Runway Gen-4.5 镜头控制最细，Seedance 2.x 批量成本最低（约 $0.01/秒）。批量化生产意味着 API 价格与一致性控制是选型关键。
4. **合规成为流水线前置环节**：中国《AI 生成合成内容标识办法》（2025-09-01 施行，抖音已强制执行）、Google Ads 2026 年 7 月上线的 AI 透明标签、欧盟 AI Act 2026 年 8 月全面执行——显式标识（"AI 生成"角标/声明）与隐式标识（C2PA/SynthID 元数据）必须写进模板，而不是事后补。

---

## 2. 脚本结构：常见范式

### 2.1 五拍式（30 秒短视频，最通用）

来源 [已验证：fotohubapp UGC 视频 campaign 文档，2026 年更新](https://github.com/fotohubapp/docs/blob/HEAD/recipes/ugc-video-campaigns.md)：

| 节拍 | 时间 | 作用 |
|------|------|------|
| Hook | 0–3s | 滚屏拦截器：视觉冲击 + 听觉反差，TikTok 要求 <2s 落核心命题 |
| Problem | 3–8s | 煽动痛点，AI 演员用"可共鸣的挫败感"验证观众现状 |
| Demo/Solution | 8–20s | 产品作为"英雄"登场；B-roll 叠加展示质地、用法、界面，配音讲 USP |
| Social Proof | 20–25s | 证言/数据/测评数/"14 天效果"陈述 |
| CTA | 25–30s | 明确指令："点黄色购物车""用码 GLOW20""主页链接下单" |

平台差异（同一来源）：TikTok 要超快节奏；Instagram Reels 偏生活化美学；YouTube Shorts 需要 Hook→Problem 之间的强留存桥段。

### 2.2 三镜式 UGC（15–20 秒，批量生产最爱）

来源 [已验证：content-factory 开源 workflow（带成本估算）](https://github.com/albertnahas/content-factory/blob/HEAD/commands/generate-ugc-ad.md)：

- **SHOT 1 — HOOK (0–5s)**：真人对镜头直述，公式如"我以前不知道……直到 [产品] 改变了这一切"；禁用品牌口号，要像真人说话。
- **SHOT 2 — DEMO (5–14s)**：讲具体转变"I used to X, now I Y"，讲"展示"时切 B-roll，结尾落在真实情绪（惊讶/松口气/兴奋）。
- **SHOT 3 — CTA (14–19s)**：朋友式推荐"说真的，试试 [品牌]"。
- 全片 80–100 词，约 18–19 秒自然语速；成本估算**约 $1.50/条**（Kling 静音头像 $0.5 + ElevenLabs 配音 $0.08 + PixVerse 口型同步 $0.5 + B-roll 等）。

### 2.3 四段式带货骨架（15 秒，中文电商语境）

来源 [已验证：dlazy-ai/ecommerce-skills product-video-ad skill（4 天前更新）](https://github.com/dlazy-ai/ecommerce-skills/blob/HEAD/skills/product-video-ad/skill.md)：

钩子 0–3s（最强视觉冲击：使用场景/痛点对比/意外角度）→ 卖点 3–9s（2–3 个特写，**一个镜头只讲一件事**）→ 场景 9–13s（真实使用场景代入）→ 落点 13–15s（商品全貌 + 价格/优惠字幕）。分镜用 JSON 描述（id/秒数/首帧图/prompt/字幕），字幕按时长自动排 SRT 时间轴。

### 2.4 四种创意方向（按目标选用）

来源 [已验证：open-drama-flow promo-video WORKFLOW（7 天前更新）](https://github.com/jiushiaaa/open-drama-flow/blob/HEAD/plugins/ai-drama-studio/skills/promo-video/WORKFLOW.md)：

- 品牌认知：情绪钩子 → 品牌价值 → 记忆点 → 品牌收束
- 产品发布：制造好奇 → 产品亮相 → 优先功能 → 证据 → CTA
- 功能演示：痛点 → 产品交互 → 可见结果 → 用户收益 → CTA
- 转化推广：问题 → 方案 → 证据 → 价值/紧迫感 → CTA

**关键设计原则**（多源一致）：先确认产品简报（≤3 个优先卖点、事实主张的证据、CTA、时长/画幅/渠道），文案按口播写，分镜覆盖完整时长无空段，每个节拍记录时间范围/旁白/画面文案/视觉动作/转场/声音意图。

### 2.5 AI 提示词公式（直接可用的模板）

来源 [已验证：cliprise awesome-ai-ugc-video-prompts](https://github.com/cliprise/awesome-ai-ugc-video-prompts)：

> 时长 + 竖屏 + 产品/服务 → 受众 → 创作者角色 → Hook(1–3s) → 问题/欲望 → 产品时刻 → Demo/证明时刻 → 镜头（手持但稳、手机感）→ 场景 → 音频 → CTA → **限制项**（无虚假宣称/无伪造证言/无标识扭曲/无文字水印/无手部畸形）

其中"限制项"字段是 2026 年社区实践的固定部分，用于压制生成模型的常见翻车（手、文字、logo 扭曲）。

**待验证/传闻**：部分教程（如 isaiahdupree 的 Veo3 UGC 教程笔记）声称单条 AI UGC 广告带来 $14,836 收入——为个人案例，无第三方佐证，不应作为 ROI 依据。来源：https://github.com/isaiahdupree/ai-video-platform/blob/HEAD/docs/video-research/V01-Z5ull80A-y0-viral-ugc-veo3-tutorial.md

---

## 3. 卖点可视化手法：AI 怎么做

### 3.1 手法清单（按产品类型）

**实体产品（电商/消费品）**——来源 [已验证：magicboat 2026-09-22 文章，抓取于 9-29](https://www.magicboat.net/mobile/en/blog/how-to-build-an-ai-ad-campaign-from-one-product-photo-script-ugc-variants-final-video)：

1. **产品真相先行**：先用文字定义产品不可变属性（材质/颜色/结构），再写脚本；一张干净的产品参照图锚定全片产品一致性。
2. **物理证明镜头**（product proof）：开盖、滴管特写、质地、涂抹、放回桌面——"让物体做点什么"，比纯展示可信。
3. **创作者场景**：真人（AI 生成的固定人设）在浴室/卧室等生活场景中手持产品，手机前置摄像头质感、轻微不完美构图、"不要影棚打光"。
4. **生活方式插入镜头**（lifestyle insert）：产品放在咖啡/书/晨光旁边，无需台词即可传达"日常感/便携/高级感"。
5. **变体不换产品**：master 资产固定，只换场景/钩子/语言/时长 → 同一产品出"干净影棚版 / 浴室 UGC 版 / 旅行版 / 夜晚版"多版本做 campaign。

**SaaS / 软件产品**——来源 [已验证：tyroneross/spectra 产品营销指南](https://github.com/tyroneross/spectra/blob/HEAD/skills/product-marketing/references/SOURCE-GUIDE.md) 与 [marcus-skills 软件 demo 研究](https://github.com/marcus/marcus-skills/blob/HEAD/research/software-demo-videos/02-best-company-examples.md)：

1. **真实录屏优先**：展示 trigger → action → output 的实时闭环；录真实工作流，用真实输出示例，不用 mock。
2. **Before/After 对比**：无插件 vs 有插件、混乱工作流 vs 整洁面板；可用分屏或硬切。
3. **"无聊功能"处理五法**：动态图形叠加 / 加速不剪（2–4x）/ 场景叙事 / 只拍好看的 / 音效设计（每次点击加清脆音效）。
4. **避免抽象 AI 意象**：发光神经网络、漂浮数据粒子——开发者和买家会直接忽略。
5. 开场钩子技巧库：哲学提问 / 痛点陈述 / 视觉奇观 / 直接点名 / 先给结果 / 社会证明数字。

**通用手法**：微距特写（材质/质地）、分屏对比、开箱流程、场景化演示（产品放进高频痛点场景，"产品成为化解冲突的必然选择"——来源 [ad-script-master](https://github.com/snailzsh/ad-script-master)）、用户证言式（AI 演员第一人称讲述 + B-roll 佐证）。

### 3.2 一致性控制（批量生产的核心技术点）

- **产品一致性**：Flux 支持多图参照（最多 8 张），同一产品换场景不变形；Seedance 2.0 支持最多 12 个参照文件（图/视频/音频/文字）。来源：[generative-tools.md](https://github.com/pedrohenriquens/claude-marketplace/blob/HEAD/plugins/marketing-skills/skills/ad-creative/references/generative-tools.md)
- **人物一致性**：Nano Banana（Gemini 图像模型）生成同一角色多角度参照，再做 image-to-video；Runway 的 reference-driven 角色一致性被评价为营销向最强。来源同上。
- **文字/logo**：Ideogram 文字渲染准确率约 90%（多数模型约 30%），广告横幅/标题文字首选它或 Nano Banana Pro。来源同上。
- **结尾卡工程**：End Card 做严格版式约束（Logo 顶部 ≤15% / 产品居中 40–50% / Slogan 底部 / CTA 对比色按钮），AI 生成时用空间坐标约束防止排版崩坏。来源：[ad-script-master](https://github.com/snailzsh/ad-script-master)

**待验证/传闻**：Seedance 2.0"每 15 秒片段约 $0.14、比竞品便宜 5–10 倍"的说法来自第三方博客估算（[heyuan110 对比文](https://github.com/heyuan110/heyuan110.github.io/blob/HEAD/content/posts/ai/2026-03-29-seedance-2-bytedance-ai-video/index.md)），官方定价以 BytePlus/即梦实时页面为准；批量化前务必实测计费。

---

## 4. 配音、字幕、多语言版本生产

### 4.1 路线总览

2026 年有两条路线并存：

- **A. 原生音频**：Veo 3.1、Kling 3.0、Seedance 2.x 生成视频时**同步生成**对白/音效/环境音。适合 UGC 自拍式口播、短镜头。
- **B. 分层制作**：静音视频（Runway、Remotion 程序化渲染）+ 独立配音（ElevenLabs/OpenAI TTS）+ 字幕（Whisper/Deepgram 转写）+ 混音。适合需要精确控制语速、情绪、品牌声线，以及**后期改词不重拍**的场景。

来源 [已验证：generative-tools.md voice 章节](https://github.com/pedrohenriquens/claude-marketplace/blob/HEAD/plugins/marketing-skills/skills/ad-creative/references/generative-tools.md)：独立配音的四种刚需——静音视频配音、品牌声线一致性、多语言版本、脚本迭代不重拍。

### 4.2 工具与选型（2026-09 现状）

| 工具 | 定位 | 多语言/关键能力 | 价格参考 |
|------|------|----------------|---------|
| ElevenLabs | 配音市场领导者 | 29+ 语言、声音克隆（即时/专业）、情绪控制、流式 API；纯音频输出，需另配剪辑 | $5/月起，约 $0.12–0.30/千字符 |
| HeyGen | 数字人 + 视频翻译 | **175+ 语言**、声音克隆、口型同步（Video Translate：同一人换语言说话）；Avatar IV 2026 更新后拟真度提升 | Creator $29/月；Business $89/月；音频配音 2026-02 起付费版无限次 |
| Synthesia | 数字人 | 120+ 语言，多语言产品介绍 | $22/月起 |
| Rask AI | 视频翻译 | 130+ 语言；口型同步锁在 $150/月 Creator Pro 档且耗 3 倍额度 | — |
| Captions.ai | 竖屏 talking-head | 自动字幕、眼神矫正、AI 配音 | — |
| CapCut | 剪辑收尾 | 免费自动字幕（多语言）、免费 TTS、模板一键成片 | 免费 / Pro $8/月 |
| Whisper / Deepgram nova-2 | 转写 | 词级时间戳，用于字幕与口型对齐 | 按量 |

来源：[HeyGen 官方 2026 横评](https://www.heygen.com/blog/heygen-vs-elevenlabs-vs-rask-ai-vs-dubverse)（注：HeyGen 自家博客，结论偏向自家产品，数据供参考）、[heygen-translate skill](https://github.com/famtastic-fritz/famtastic-designs/blob/HEAD/.agents/skills/heygen-translate/SKILL.md)、[fahimai HeyGen 评测中文版](https://www.fahimai.com/zh/heygen)、[kangise TikTok Shop AI 指南](https://github.com/kangise/ecommerce-ai-skills/blob/HEAD/src/d-platforms/tiktok-shop-ai-guide.md)。

### 4.3 多语言批量生产的推荐工作流

综合 [ryviuszero 多语言工作流](https://github.com/ryviuszero/voice-tools/blob/HEAD/src/content/workflows/youtube-multilingual.md) 与社区实践：

1. 先拿**源语言转写**，清理口误和重复 → 再翻译脚本（保留术语表、品牌名、不可翻译词）。
2. **新建视频**（无源片）：直接用目标语言写脚本，走 heygen-video / ElevenLabs 生成——不要先做源语言片再翻译。
3. **已有成片**：HeyGen Video Translate 做声音克隆 + 口型同步；或纯音频配音（ElevenLabs）+ 换音轨。
4. 字幕：CapCut/Whisper 自动生成 → 人工校对 → 按目标语言重新排版（德语/法语文本更长，安全区要留余量）。
5. 母语者抽查：开头 30 秒、价格/数据、CTA、易误解段落必须人工听一遍。
6. 先做 2–3 个已有需求的语言验证（看 Analytics 的海外观看/评论语言），不要一次铺十种语言。

**免费/低成本路径**（待验证细节，来源为 2 天前博客 [lilachbullock.com](https://www.lilachbullock.com/translate-videos-with-ai-for-free/)）：YouTube Studio 自动配音（符合条件频道免费）、ElevenLabs 免费档（约 1 万字符/月，只够 1–2 条短视频）、CapCut 免费字幕+TTS。免费档额度经常调整，规划批量生产前需核实。

---

## 5. 常见工具链（脚本 → 成片）

### 5.1 2026-09 视频生成模型格局

综合 [futurelume 2026-09 对比](https://futurelume.net/reviews/product-comparisons/best-ai-video-generators-in-2026-veo-vs-kling-vs-seedance-vs-sora/)、[convly 2026-09-29 更新](https://convly.ai/best-ai-video-generators-2026/)、[avocadoai 2026-07 对比](https://www.avocadoai.co/blog/comparison/ai-video-generator-features-comparison-2026)：

| 模型 | 最擅长 | 原生音频 | 分辨率/时长 | API 价格量级 |
|------|--------|---------|------------|-------------|
| Google Veo 3.1 | 营销向全能、提示词服从度最高 | 有 | 4K（upscale)/60s | $0.05–0.40/秒 |
| Kling 3.0 | 电影感、高动态多镜头 | 有（Voice Binding) | 4K/最长 3 分钟(2.6) | $0.07–0.09/秒 |
| Runway Gen-4.5 | 镜头控制、角色一致性、后期编辑 | 弱/无 | 1080p–4K/10s | 订阅制 |
| Seedance 2.0/2.5（字节） | 高性价比批量、分镜驱动 | 有（双分支联合生成） | 2K/15–20s | 约 $0.01–0.10/秒 |
| Pika | 社交特效、快速竖屏 | 有 | — | — |
| Higgsfield | 聚合器：多模型 + 50+ 电影运镜 + 剪辑 | 依模型 | 1080p | $15–49/月 |
| Sora 2 | — | — | — | ⚠️ **API 已于 2026-09-24 关停**，App 早在 4 月关闭；存量工作流必须迁移 |

**2026 年核心方法论**（futurelume 原话）："Best is the wrong question — ask 'best for this shot.'" 按**镜头**选模型，不按项目选。Seedance 提示词必须用中文、单次 4–15 秒、超 15 秒用"视频延长"拼接且段间要有画面衔接点（[ad-script-master](https://github.com/snailzsh/ad-script-master)）。

### 5.2 三条典型链路（社区实际在用）

**链路 A：纯 AI UGC 投流素材（电商向）**
ChatGPT/Claude 写脚本 → Nano Banana 生成固定角色图 → Veo 3/Kling 生成 A-roll（自拍口播）+ B-roll（产品使用）→ ElevenLabs 配音（如需）→ CapCut 自动字幕 + 剪辑成片。
来源：[ai-video-platform UGC 教程笔记](https://github.com/isaiahdupree/ai-video-platform/blob/HEAD/docs/video-research/V01-Z5ull80A-y0-viral-ugc-veo3-tutorial.md)

**链路 B：Agentic 广告平台（一站式）**
AdCraft（开源，2026 年活跃）：创意规划 → 脚本 → 产品视觉 → 角色 → 场景 → 分镜 → 资产生成 → 剪辑 → 成片，全流程可编辑画布，支持 Seedance/Kling/Vidu 多模型。来源：[gml-mmgroup/adcraft](https://github.com/gml-mmgroup/adcraft)
MagicBoat：脚本 → 分镜 → 视觉 → 配音 → 剪辑 → 成片，强调并行版本与可复用资产。来源：[magicboat 文章](https://www.magicboat.net/mobile/en/blog/how-to-build-an-ai-ad-campaign-from-one-product-photo-script-ugc-variants-final-video)
Higgsfield（2026-09-21 发布 agentic 广告工作流指南）：创意研究 → 脚本 → 生产 → 版本化 → 发布 → 效果分析连成一条链。来源：上文引用的 https://higgsfield.ai/blog/claude-ad-production-end-to-end（未直接抓取，为二手引用，待验证）

**链路 C：程序化/代码驱动（批量与个性化）**
Remotion（React 写视频）：脚本/数据 → 模板化场景 → 与配音对齐 → 渲染。适合价格/名称/语言变量的**千条级**个性化与品牌强一致场景；AI 生成 B-roll 做氛围镜头，真人/录屏做产品主体。来源：[storyboard SKILL](https://github.com/matt-j-penny/ai-video-marketing/blob/HEAD/skills/video-production/storyboard/SKILL.md)、[flex-kit marketing-video](https://github.com/haonguyen1915/flex-kit/blob/HEAD/flex_kit/packs/disciplines/marketing/skills/marketing-video/SKILL.md)

**链路 D：低成本人工混合（TikTok Shop 实战）**
"1 人 1 天 5 条"：AI 写 5 条脚本（10 分钟）→ 手机实拍产品/场景素材（一次拍摄剪 10+ 条）→ CapCut AI 剪辑/字幕/配音（15 分钟/条）→ 发布。纯 AI 路线（HeyGen 数字人 + ElevenLabs + CapCut 合成）适合标品，劣势是真实感不如真人。来源：[kangise TikTok Shop AI 指南](https://github.com/kangise/ecommerce-ai-skills/blob/HEAD/src/d-platforms/tiktok-shop-ai-guide.md)

### 5.3 确定性组装层

多源一致：**FFmpeg 是最终合成的事实标准**（拼接、重编码、烧录字幕）；分镜 JSON → 按秒数自动排 SRT 时间轴 → ffmpeg 串成片，编码不一致自动重编码。来源：[dlazy-ai product-video-ad](https://github.com/dlazy-ai/ecommerce-ai-skills/blob/HEAD/skills/product-video-ad/skill.md)、[open-drama-flow](https://github.com/jiushiaaa/open-drama-flow/blob/HEAD/plugins/ai-drama-studio/skills/promo-video/WORKFLOW.md)。CapCut 则承担"最后 10%"的平台化包装（热门 BGM、平台字幕样式、比例裁切）。

---

## 6. 合规与标识（2026 年批量生产必须前置）

来源 [已验证，多源交叉]：

1. **中国**：《人工智能生成合成内容标识办法》2025-09-01 施行（注：部分报道写作"2026年9月1日"，实为法规 2025 年 9 月 1 日施行，2026 年 9 月抖音等平台强化执行——引用时注意区分）。要求：显式标识（显著位置"AI 生成"声明）+ 隐式标识（元数据，含生产者身份）；广告含 AIGC 须声明"本广告使用 AI 技术"/"本广告由 AI 技术生成"，数字人须单独声明并与真人区分；用真人形象做数字人须书面授权。抖音：未标识单条会被标注，累计 5 条以上限流；数字人账号须实名；2026 年一季度处置超 80 万条 AIGC 违规带货内容。来源：[baize-website 笔记](https://github.com/smartstudio/baize-website/blob/HEAD/site/src/content/notes/ai-video-ads-vs-brand-review.mdx)、[新浪新闻](https://k.sina.cn/article_7879995946_1d5af322a06801qxoa.html?from=tech)
2. **Google Ads**：2026-07-09 上线 AI 透明标签，"My Ad Center" 新增"How this ad was made"披露；支持 SynthID + C2PA 元数据；欧盟/印度/纽约等地区要求画面可见标签。来源：上同 baize 笔记（转引 Google 官方博客）。
3. **欧盟**：AI Act 2026-08-02 全面执行，deepfake 必须标注、生成式 AI 须嵌入机器可读水印；罚则最高 €3500 万或年营收 7%。来源：[yuyangzi 内容治理文章](https://github.com/yuyangzi/contentcreationkit/blob/HEAD/content/article/20260719-AI内容治理强制标注时代.md)（作者分析向，非官方文件）
4. **平台违禁词**（中文投流）：极限词（最/第一/最佳）、"揭秘/内幕"、医疗词（治愈/根治）、金融承诺、"100% 有效/零风险"；字幕须与口播 100% 一致；镜头中 logo/二维码/微信号会限流。来源：[llucas88 stable skill](https://github.com/llucas88/stable/blob/HEAD/desktop/skills/filtered/bundle/packages/hub-e0f69de3/SKILL.md)

**对流水线的含义**：baize 笔记提出的"先过审口径，再开批量"——禁词表、标识角标位置、肖像授权清单全部写进模板；疗效承诺、before/after 对比、真人肖像/声音、数字人口播**必须人工复核**；转码后检查隐式元数据是否丢失（截屏/压缩/二次编码会丢水印）。

---

## 7. 信息时效声明

- **2026-09（可信度高）**：Veo 3.1/Kling 3.0/Runway Gen-4.5/Seedance 2.5 格局；Sora API 2026-09-24 关停；Google Ads AI 标签（2026-07）；欧盟 AI Act 全面执行（2026-08）；Higgsfield agentic 广告工作流（2026-09-21）；抖音 AIGC 标识强化执行（2026-09）。
- **2026 年中（基本可用，注意价格/额度变动）**：各工具定价（ElevenLabs、HeyGen、CapCut）、HeyGen 2026-02 音频配音无限次、Seedance 成本估算。
- **2025 及更早（仅作范式参考）**：ChatGPT 式录屏 demo 方法论、部分 Medium 电商文章（2026-01 发布但观点偏营销软文，其"转化提升 210%"等数据为**单方宣称**，不可引用为行业数据）。
- **二手引用（未直接抓取原文）**：Higgsfield 官方博客 agentic 工作流指南，仅见于 magicboat 文章引用，细节待验证。

---

## 8. 对 video-factory-lab 营销视频线的设计建议

1. **脚本层做成"结构模板 × 变量槽位"**：固化 3 套骨架（30s 五拍式 / 18s 三镜 UGC / 15s 四段带货），变量只有受众、痛点、卖点（≤3）、CTA、语言。LLM 只填槽位不发明结构；每条脚本附带"限制项"字段（无虚假宣称/无手部畸形/无文字水印），直接拼进视频 prompt。
2. **资产层先建"产品真相库"**：每个产品一条 product-truth 记录（文字属性 + 1 张以上干净参照图 + 品牌色/字体/禁词表/授权清单）。所有镜头生成都引用它；人物用固定 AI 人设 + 多角度参照图。这是批量不变形、不过审的根基。
3. **生成层按镜头路由模型**：口播/A-roll 用 Veo 3.1 或 Kling（原生音频），产品特写/批量 B-roll 用 Seedance（成本低），精细运镜用 Runway，文字多的结尾卡/横幅用 Ideogram 或 Nano Banana Pro 生成底图再合成。**不要把 Sora 写进任何新链路**（API 已关停）。
4. **组装层用确定性工具收口**：分镜 JSON（含每镜秒数、字幕）→ ElevenLabs/CapCut TTS 配音 → Whisper 对齐字幕 → FFmpeg 拼接/烧录。AI 只负责"镜头内容"，时长、字幕时间轴、标识角标位置由代码保证——这是模板化可复制的关键。
5. **合规与变体做成流水线内置步骤**：模板里预置 AI 标识角标位置与声明文案（国内外两套）；发布前跑禁词/极限词扫描 + 人工复核清单（疗效承诺、before/after、真人肖像）。变体策略固定为"换钩子/换场景/换语言/换时长"四轴，master 资产复用，一次生产出 4–8 条做 A/B 测试。
