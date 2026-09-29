# 专题1：2026 年 AI 视频生成市场格局研究报告

> **调研日期：2026-09-29**
> **调研方式：** 通过公开网络搜索（browser.search）与页面文本抓取（browser.open），覆盖中英文来源。未做实时浏览器登录/付费验证，所有价格与配额均来自公开资料，抓取时均标注来源与抓取时间。
> **信息时效声明：** 优先采用 2026 年 Q2–Q3 的资料（发布/更新时间在每条结论后标注"截至"）；2024–2025 年的旧资料仅作背景参考并明确标注。本报告结论的有效期按"月"计，价格类信息建议每月复核。

---

## 0. 一句话总览：2026 年的三件大事

1. **Sora 已死。** OpenAI 于 2026-04-26 关闭 Sora 消费端（App/Web），并于 **2026-09-24 正式关闭 Videos API**（sora-2 / sora-2-pro 全系下线），且未指定任何替代模型（截至 2026-09-29）。这是 2026 年视频 API 市场最大的"反面教材"：单模型依赖 = 单点故障。
   （来源：https://magichour.ai/blog/what-is-sora ；https://mediovsky.com/sora-api-shutdown/ ；https://www.knowmouth.com/openai-sora-shutdown-sora-2-api-discontinued）
2. **中国模型成为事实上的质量+成本双标杆。** Seedance 2.5（字节，2026-06-23 发布）实现单段原生 30 秒、原生 4K、50 个多模态参考；MiniMax H3（2026-07-31 发布，8 月开源权重）以 33B 开放权重 + 原生立体声音频冲击市场，迫使 Seedance 在 8–9 月首次出现渠道折扣（2.0 最低 5.5 折）。
   （来源：https://github.com/ethanniworld/ai-playbook-2026/blob/HEAD/knowledge/bytedance/seedance-series.md ；https://www.21jingji.com/article/20260916/herald/8f990c6444eb701692731b4e72375e0d.html ；https://www.technology.org/2026/09/07/ai-videos-new-power-trio-seedance-2-5-minimax-h3-wan-3/）
3. **"原生音频"成为 2026 年的标配分水岭。** Veo 3.1、Kling 3.0、Seedance 2.5、Wan 3.0、Hailuo H3、LTX-2.3 全部支持音画同出；开源侧 LTX-2.3 是唯一单次扩散同时生成音视频的开放模型。音频策略（原生 vs TTS 后期）直接影响流水线架构。
   （来源：https://github.com/juspay/director/blob/HEAD/video-production/library/docs/VIDEO-GEN-LANDSCAPE-2026Q1.md ；https://www.thundercompute.com/blog/best-open-source-ai-video-generation-models）

---

## 1. 商用模型 / API 格局

### 1.1 梯队划分（截至 2026-09）

| 梯队 | 模型 | 一句话定位 |
|---|---|---|
| T0 旗舰 | Seedance 2.5（字节）、Veo 3.1（Google） | 质量天花板；Seedance 长镜头+多参考，Veo 音频+4K+生态 |
| T1 主力 | Kling 3.0（快手）、Runway Gen-4.5/Aleph、Wan 3.0（阿里） | 各有绝活：Kling 运动流畅度、Runway 编辑可控、Wan 多模态参考 |
| T2 高性价比 | Hailuo H3（MiniMax）、PixVerse V6、Vidu Q3、Seedance 2.0 Mini | 量产/迭代的主力弹药 |
| 专用/利基 | Luma Ray 3.2、Pika 2.5、Hunyuan（腾讯）、即梦系 | 物理仿真、快速原型、中文生态 |
| ☠️ 已退役 | Sora 2 / Sora 2 Pro | API 已于 2026-09-24 关闭，勿再选型 |

### 1.2 各家详情

**Google Veo 3.1（截至 2026-09）**
- 规格：单段 4/6/8 秒，720p/1080p/4K（唯一原生 4K 的商用模型之一），24fps（4K 可达 60fps），原生同步对白/音效/环境音，SynthID 隐形水印（不可关闭）。
- 2026 年 1 月更新：原生 9:16 竖屏、"Ingredients to Video"（最多 4 张参考图）、角色一致性增强；支持 Scene Extension 可拼接至 60 秒+。
- API：Gemini API（`veo-3.1-generate-preview`，`predictLongRunning` 异步端点）+ Vertex AI，均已开放。另有 Fast / Lite 低价档：Lite 仅为 Standard 的约 1/8 价格。
- 价格（按生成秒计）：Standard $0.40/s（含音频），Fast $0.15/s，Lite $0.03–0.05/s；4K 为 $0.35–0.60/s。8 秒 1080p 标准档约 $3.20，Fast 档约 $1.20。
- 注意：第三方文档提到 Veo 3.1 的 GA 版本标注了 **2026-11-17 或更晚的退役日期**（待官方确认，见 §5）。
- 来源：https://github.com/juspay/director/blob/HEAD/video-production/library/docs/VIDEO-GEN-LANDSCAPE-2026Q1.md ；https://github.com/kacky000/aitoolpick/blob/HEAD/src/content/blog/google-veo-pricing-2026.md ；https://mediovsky.com/sora-api-shutdown/

**Runway Gen-4.5 / Gen-4 Aleph / Act-Two（截至 2026-09）**
- Gen-4.5：当前主力，文生+图生视频，约 10 秒，1080p。API 按秒计费 **$0.12/s**（Turbo 档 $0.05/s）；订阅 Standard $12/月（625 credits，年付）起。官方建议 Turbo 打草稿、Gen-4.5 出成片。
- Gen-4 Aleph：视频到视频精准编辑（换装/换景/换天气/风格化），API 15 credits/s（$0.15/s）；Aleph 2.0 走 Edit Studio 工作流。第三方聚合标注 Aleph 2 API 约 $0.336/s。
- Act-Two：表演迁移（真人表情/动作迁移到静态角色），约 $0.05/次（5 credits/s）。
- 来源：https://github.com/heygen-dev/runway-gen-4/blob/HEAD/README.md ；https://www.aixploria.com/en/runway-gen-4-5/ ；https://vidofy.ai/en/models/runway/gen-4-aleph ；https://www.therundown.ai/tools/aleph

**Kling 3.0（快手可灵，截至 2026-09）**
- 规格：1080p，5–10 秒，原生音频；Omni 变体支持多图参考驱动的编辑/生成；Pro Motion Control 端点（复杂电影运镜，约 $0.16/s）。
- 订阅：免费档每日 66 credits；Standard $6.99/月（年付约 $8.8/月续费价有浮动报告）。
- API：官方开发者平台（kling.ai/dev/pricing）+ 第三方聚合。fal 口径：Standard $0.084/s（无音频）/$0.126/s（含音频），Pro $0.112/s / $0.168/s（含音频，+人声 $0.196/s）。
- 注意：官方定价页曾被报告"无法读取当前报价"，第三方价不能替代官方报价；媒体称其海外收入占比约七成。
- 来源：https://github.com/rrrrrredy/research-toolkit/blob/HEAD/evals/diagnostics/2026-09-07/reports/video-production/reviews/before-first-source-review.md ；https://www.fahimai.com/zh/kling-ai ；http://www.360doc.com/content/26/0216/00/57798620_1170117513.shtml ；https://news.marsbit.co/20260923163210637160.html

**Seedance 2.5 / 2.0（字节跳动，截至 2026-09）**
- 2.5（2026-06-23 FORCE 大会发布）：单段**原生 30 秒**直出、原生 4K（2026 年 6 月升级）、**最多 50 个全模态参考素材**（角色/场景/道具/音色）、音画同出；工业级多镜头叙事，被视为 AI 短剧生产的当前最强项。
- 定价（火山方舟，token 口径，第三方转述 2026-08 核实）：2.5 纯文生约 70 元/百万 tokens，折合约 **720P 1.51 元/秒、480P 0.67 元/秒**；2.0 约 720P 1 元/秒（官方口径经 36 氪转述，"从不打折"但 2026-08/09 起渠道出现折扣）；2.0 Mini 约 0.5 元/秒（成本砍半）。
- API：火山方舟（国内）+ BytePlus ModelArk（出海）；国际聚合 fal 上 2.5 @480p 约 $0.2205/s、@720p 约 $0.4730/s（约 2 倍于直连价的溢价）。
- 动态：钛媒体 2026-09-16 报道，受 MiniMax H3 冲击，Seedance 渠道 8–9 月开始主动谈折扣（2.0 最低 5.5 折、2.5 可 8.8 折），视频赛道进入"降价换调用量"阶段。
- 来源：https://github.com/ethanniworld/ai-playbook-2026/blob/HEAD/knowledge/bytedance/seedance-series.md ；https://github.com/moemu/richidrama/blob/HEAD/docs/plans/2026-08-10-volcengine-official-pricebook.md ；https://www.21jingji.com/article/20260916/herald/8f990c6444eb701692731b4e72375e0d.html ；https://www.eet-china.com/mp/a503020.html

**Luma（Ray 3.2，截至 2026-09）**
- "Dream Machine" 已退役并入 **Luma App**，当前视频模型为 **Ray 3.2**（另有 Ray3.14 的提法，见 §5 待验证）；5/10 秒片段，1080p 上限，强项是物理/空间理解（产品可视化、建筑、自然场景），支持 HDR 输出（第三方评测口径）。
- 订阅：Plus $30/月（10,000 credits）、Pro $90/月（40,000）、Ultra $300/月（150,000），商用授权从 Plus 起；免费入口存在但额度不再公开。
- API 独立计费（Build 档默认 $5,000/月上限，失败生成退款——订阅端失败不退款）；Ray 2 时代 API 约 $0.95/5 秒 1080p。
- 关键限制：**Luma 未上任何主流聚合器**（fal/Replicate/Runware 均无），只能官方直连。
- 来源：https://www.eesel.ai/blog/luma-ai-pricing ；https://magichour.ai/blog/luma-dream-machine-pricing ；https://github.com/madponyinteractive/cubric-vision/blob/HEAD/docs/proprietary-models-research/01-aggregators.md

**Pika 2.5（截至 2026-09）**
- 定位：最快（单片段 8–15 秒）、Pikaffects 病毒特效、Pikaformance 口型同步、Pikaframes 最长 25 秒插值；1080p 上限，10 秒标准。
- 订阅：免费 80 credits/月（480p），Standard $10/月（700）、Pro $35/月（2,300）、Fancy $95/月（6,000）；免费档水印与商用授权政策各来源说法不一（以 pika.art 实时页面为准）。
- API：**官方 API 有限——2.2 经 fal.ai 提供（约 $0.20/条 720p、$0.45/条 1080p），2.5 暂无 API**。对流水线而言 Pika 主要是"快速原型/特效"角色。
- 来源：https://magichour.ai/blog/pika-labs-pricing ；https://github.com/mua47105-hue/agentic-ai-video-knowledgebase/blob/HEAD/kb/wiki/archive/pika-2-5.md ；https://github.com/api-evangelist/pika

**Hailuo H3（MiniMax，截至 2026-09）**
- 2026-07-31 发布，**2026-08-03 开源 Base 权重**：33B dense 单流 Transformer，文本/首尾帧/多模态参考（最多 9 图 + 3 视频 + 3 音频）输入，**原生 32kHz 立体声音频**，4–15 秒，本地 768p，官方完整工作流可再生至 2K。
- API：MiniMax 开放平台 `MiniMax-H3`；fal 上约 **$0.26/s**（另有 fal 8-27 推出的后训练衍生版 H3 Max）。
- 市场影响：开源+低价直接冲击 Seedance 的广告/2D 等价格敏感客户（钛媒体 2026-09-16）。
- 来源：https://github.com/gvclab/video-generation-101/blob/HEAD/sources/research_20260829_minimax_h3.md ；https://github.com/anastasiyaw/knowledge-space/blob/HEAD/docs/projects/minimax-h3.md ；https://github.com/localsymmetry/lofn/blob/HEAD/output/analysis/2026-08-15_video-model-landscape.md

**Vidu Q3（生数科技，截至 2026-09）**
- Q3 模型支持 **16 秒音视频同步生成**、多镜头合成与镜头控制；据报阿里云领投 20 亿元融资用于世界模型研发。
- API：聚合器 MuAPI 上 Vidu Q3 Pro 约 **$0.04/次**（1080p），是独立榜单中价格最低的高质量档之一；国内智谱平台亦有 Vidu2（720p 低成本）/ViduQ1（1080p 影视级）等多档。
- 来源：https://www.stnn.cc/c/2026-04-10/4055997.shtml ；https://github.com/anil-matcha/awesome-ai-video-models ；https://github.com/ht-shaipe/soma/blob/HEAD/docs/china-ai-video-api-research.md

**Wan 3.0（阿里，截至 2026-09）**
- 2026-08-06 公开 beta：单模型内统一文生/图生/首尾帧/参考生成，**最长 30 秒 @30fps**，480p/720p/1080p，同步音频，支持最多 20 个多模态参考（含文档→视频）。
- API：阿里云 Model Studio / Qwen 平台、Vercel AI Gateway（`alibaba/wan-v3.0-video`）；Wan 2.7 另有**音频驱动口型同步**模式（`audio_url` 参数传入配音，模型锁口型），对漫剧对白是杀手特性。
- 来源：https://vercel.com/changelog/wan-3-0-now-available-on-ai-gateway ；https://github.com/civitai/civitai-developer-docs/blob/HEAD/orchestration/recipes/wan.md ；https://github.com/atlascloudai/awesome-wan-3.0-prompts ；https://github.com/sutchan/agent-skills-hub/blob/HEAD/skills/ai-video-generation-runcomfy/SKILL.md

**其他（截至 2026-09，简表）**
- PixVerse V6：fal 上 $0.033（360p 无音频）– $0.150（1080p 含音频）/s，按分辨率阶梯计价，低成本量产之选；据报用户规模达亿级。
- 即梦（Dreamina，字节）：Seedance 2.5 的消费端入口之一；视频 API 主要走火山方舟/即梦开放体系，国内按 token 计费。
- Hunyuan（腾讯混元）：API 成熟度中等，1–3 元/次口径；开源 HunyuanVideo 见 §3。
- 来源：https://github.com/anil-matcha/awesome-ai-video-models ；https://github.com/ht-shaipe/soma/blob/HEAD/docs/china-ai-video-api-research.md ；https://news.marsbit.co/20260923163210637160.html

### 1.3 名义生成成本对比（8 秒 1080p，fal/官方口径，截至 2026-09）

| 模型 | 单价口径 | 8 秒名义成本 | 备注 |
|---|---|---|---|
| Veo 3.1 Lite | $0.05/s | **$0.40** | 质量档位最低，音频原生 |
| Veo 3.1 Fast | $0.15/s | $1.20 | |
| Kling 3.0 Pro（含音频） | $0.168/s | $1.34 | fal 口径；直连价更低但难读 |
| Veo 3.1 Standard | $0.40/s | $3.20 | 4K 更贵（$0.35–0.60/s）|
| Seedance 2.5（fal @720p）| ~$0.473/s | ~$3.78 | 直连约一半价；单段可达 30s |
| Runway Gen-4.5 | $0.12/s | $0.96 | Turbo $0.05/s → $0.40 |
| Hailuo H3 | $0.26/s | $2.08 | 2K 档；开源可本地降本 |
| Vidu Q3 Pro | ~$0.04/次 | ~$0.04 | 按次计费，极低 |
| PixVerse V6 1080p | $0.150/s | $1.20 | 360p 仅 $0.26/8s |

> ⚠️ 名义成本 ≠ 实际成本。一份 2026-09-07 的中文测算研究指出，应以**"每个采用秒的成本 =（生成+重试+修正+剪辑+音频+审核）/ 实际采用秒数"**核算：若 Kling 采用率 20%、Veo 采用率 50%，两者每采用秒成本几乎打平（$0.84 vs $0.80）。**采用率（一次过审率）是比单价更重要的成本变量。**
> （来源：https://github.com/rrrrrredy/research-toolkit/blob/HEAD/evals/diagnostics/2026-09-07/reports/video-production/reviews/before-first-source-review.md）

---

## 2. API 接入与聚合器生态

### 2.1 官方 API 开放情况（截至 2026-09-29）

| 厂商 | 官方 API | 状态 |
|---|---|---|
| Google Veo 3.1 | ✅ Gemini API + Vertex AI | 开放，按秒计费 |
| Runway | ✅ Runway API（独立账单） | 开放，$0.01/credit |
| Kling | ✅ 官方开发者平台 + 聚合 | 开放（定价页可读性曾被诟病） |
| 字节 Seedance | ✅ 火山方舟（国内）/ BytePlus ModelArk（出海） | 开放；满血版有年框门槛传闻 |
| Luma | ✅ Luma API | 开放，但**不上聚合器** |
| Pika | ⚠️ 有限 | 2.2 经 fal 间接提供；2.5 无 API |
| MiniMax H3 | ✅ MiniMax 开放平台 | 开放 |
| 生数 Vidu | ✅ 智谱/自有开放平台 + 聚合 | 开放 |
| 阿里 Wan | ✅ Model Studio / Qwen | 开放 |
| OpenAI Sora | ❌ | **2026-09-24 已关闭** |

### 2.2 聚合器（对流水线最重要的中间层）

- **fal.ai**：视频 API 聚合市占率约 44%（第三方调研口径）；统一 JSON schema、按秒透明计价、queue+webhook+5xx 不计费；Kling/Veo 与官方**零差价**（fal 赚厂商分成），Seedance 约 2 倍溢价但对海外个人/小公司是最低摩擦的官方授权渠道；Kling 3.0、Hailuo 2.3 均为 day-0 上线。**注意：fal 上无 Runway、无 Luma。**
- **Replicate**：按次计费、无订阅，托管大量开源模型（Wan 2.1/2.2 480p 极便宜、LTX 2.5、PixVerse v4），也有 Veo 3.1 Lite（$0.05/s）、Kling v3 等。
- **WaveSpeed AI**：600+ 模型单 API，无排队/地域限制；Kling 3.0 Standard $0.084/s、Veo 3.1 Fast $0.15/s。
- **Runware**：按条（per clip）计费；是**唯一有 Runway Gen-4.5** 的聚合器（$0.605/条）；无 Luma。
- **MuAPI**（中文聚合）：Seedance 2.5、Vidu Q3 Pro 等国产模型价格极低（Vidu Q3 Pro $0.04/次）。
- **BytePlus ModelArk**：Seedance 出海官方直连，约为 fal 一半价，个人可开（180RPM/3 并发）。
- 来源：https://github.com/madponyinteractive/cubric-vision/blob/HEAD/docs/proprietary-models-research/01-aggregators.md ；https://github.com/madponyinteractive/cubric-vision/blob/HEAD/docs/proprietary-models-research/01b-more-aggregators.md ；https://github.com/bthurlow/artificer-mcp/blob/HEAD/docs/plans/fal-ai-integration.md ；https://github.com/belcort-sdn-bhd/fikirtive/blob/HEAD/docs/research/genapi-provider-research.md

---

## 3. 开源模型

### 3.1 对比总表（截至 2026-09）

| 模型 | 机构 | 参数 | 最低 VRAM | 分辨率×时长 | 原生音频 | 许可（商用） | 社区热度 |
|---|---|---|---|---|---|---|---|
| Wan 2.7 / 3.0 | 阿里 | —/— | 8GB（GGUF/量化） | 1080p × 15s / 30s | 2.7 支持音频驱动口型；3.0 同步音频 | Apache 2.0 | ★★★★★ |
| LTX-2.3 | Lightricks | 22B | 8–12GB（蒸馏版） | 4K × 20s | ✅ 单次扩散音画同出（开源唯一） | 商业需授权（年收入 >$10M） | ★★★★★ |
| HunyuanVideo 1.5 | 腾讯 | 8.3B | ~14GB（FP8+offload） | 720p × 10s+ | ❌ | 社区许可（<1 亿 MAU 可商用） | ★★★★ |
| MiniMax H3 Base | MiniMax | 33B | 消费级可跑（官方口径） | 768p × 4–15s | ✅ 原生立体声 | 开放权重 | ★★★★（新） |
| CogVideoX | 智谱/清华 | 2B/5B/1.5 | 12–24GB | 720p × 6–10s | ❌ | Apache 2.0（2B）/开放权重 | ★★★★ |
| Mochi 1 | Genmo | ~10B | ~20GB（FP8） | 480p/720p × 5s | ❌ | Apache 2.0 | ★★★★ |
| Open-Sora 2.0 | HPC-AI Tech | 11B | 24GB+ | 多规格 | ❌ | Apache 2.0 | ★★★ |
| FramePack | lllyasviel | 依 base | 依 base | 1080p × 无限循环 | ❌ | 开源 | ★★★★ |
| SVD | Stability | — | 16GB | I2V 为主 | ❌ | NC（非商用研究）⚠️ | ★★★ |

> 许可警告：SVD 为 Stability 非商用（NC）许可，**不可直接用于商业流水线产出**；LTX-2.3 年收入超 $10M 需商业授权；HunyuanVideo 1.5 超 1 亿 MAU 受限。Wan、Mochi 1、CogVideoX-2B 为 Apache 2.0，无商用限制（以各仓库 LICENSE 为准）。
> （来源：https://www.thundercompute.com/blog/best-open-source-ai-video-generation-models ；https://github.com/seanwinslow28/anima/blob/HEAD/docs/open-sourced-video-model-research/Open-Source-Video-Models-for-ComfyUI-Gemini.md ；https://github.com/ishandutta2007/video-generation-landscape/blob/HEAD/README.md）

### 3.2 分述

- **Wan（阿里，Apache 2.0）**：2026 年开源视频的事实标准。Wan 2.2 有 5B GGUF 版可跑 8GB 显存；**Wan 2.7（2026-04）引入音频驱动模式**——传入 `audio_url`，模型按波形锁口型，TTS 与视频解耦，解决"模型自带音频不可控"痛点；**Wan 3.0（2026-08）** 30 秒/30fps/1080p + 同步音频 + 20 资产多模态参考。ComfyUI 原生支持极好（kijai WanVideoWrapper）。
- **LTX-2.3（Lightricks）**：速度之王（蒸馏版 8 步出草稿）+ **开源唯一的单次音画同出**；IC-LoRA 支持深度/姿态/边缘/运镜控制；Apple MLX 原生支持；**已内置进 ComfyUI 核心**，官方 ComfyUI-LTXVideo 节点包。代价：22B 全量版需 24GB+，画质上限不如 Wan/Hunyuan。
- **HunyuanVideo 1.5（腾讯）**：8.3B，当前开源画质 SOTA 之一（第三方口径）；FP8 + CPU offload 约 14GB 可跑；中文社区有大量动漫 LoRA。
- **CogVideoX（智谱/清华）**：2B/5B/1.5 三档，CogKit 微调框架成熟，ComfyUI wrapper 活跃；智谱云上另有 CogVideoX-3（4K，约 1 元/次）。
- **Mochi 1（Genmo，Apache 2.0）**：物理真实感好，官方单卡 LoRA 训练器，适合做风格化定制。
- **Open-Sora 2.0**：11B，附完整训练管线，偏研究向，生产用得少。
- **FramePack**：首尾帧精确匹配 + 无限循环生成，2D 动画/循环素材神器，有 kijai 的 ComfyUI 封装。
- 来源：https://medium.com/@jon_davis/the-open-source-ai-video-stack-in-2026-what-actually-works-9b20fb281ec2 ；https://github.com/quriosity-agent/articles/blob/HEAD/2026-03-18/chris-defi-2033832631083966913-analysis-en.md ；https://github.com/septa-serpenta-seraph/vesper/blob/HEAD/skills/.archive/ltx23-comfyui-setup/SKILL.md

### 3.3 硬件门槛速查（单卡）

| 档位 | 显存 | 能跑什么 |
|---|---|---|
| 入门 | 8GB | Wan 2.2 5B（GGUF）、LTX-2.3 蒸馏版、AnimateDiff |
| 主流 | 16GB | CogVideoX-2B、LTX-2.3、SVD |
| 专业 | 24GB（RTX 4090） | Wan 14B、Mochi 1、HunyuanVideo（GGUF/FP8）、LTX-2.3 全量 |
| 云端 | 40–80GB | HunyuanVideo 全精度、LTX-2.3 4K+音频、批量并发 |

云 GPU 参考价：Thunder Compute 上 ComfyUI 实例 RTX A6000 48GB 约 $0.35/小时；Wan 2.2 A14B 出 480p 片段约 $0.02–0.03/条，远低于商用 API 量产成本。
（来源：https://www.thundercompute.com/blog/best-open-source-ai-video-generation-models ；https://github.com/dan-ritual/video-ai-primer/blob/HEAD/06_COMFYUI_NODE_WORKFLOWS_GUIDE.md）

---

## 4. ComfyUI 生态在视频生成方面的现状（截至 2026-09）

### 4.1 官方/核心集成（拐点：视频节点正在"转正"）

- **LTX-2 已内置进 ComfyUI 核心**，Lightricks 官方维护 `ComfyUI-LTXVideo` 节点包（文本编码、采样、IC-LoRA、音视频潜变量分离/合并、Lipdub 口型等），随 ComfyUI Manager 一键安装。
- **Wan 系**：kijai 的 `ComfyUI-WanVideoWrapper` 是事实标准（支持 Wan 2.1/2.2/2.6/2.7，T2V/I2V、MoE）。
- **FramePack**：注意 `lllyasviel/FramePack` 是独立 Gradio 应用，ComfyUI 内用 `kijai/ComfyUI-FramePackWrapper`。
- 另有 MiniMax H3 的社区节点（如 `comfyui-minimax-h3-audio-T8`，分离音视频潜变量）已在 2026-08 出现。
- 来源：https://github.com/Lightricks/ComfyUI-LTXVideo/ ；https://github.com/joshwaamein/joshwaamein.github.io/blob/HEAD/_posts/2026-04-05-comfyui-amd-rx7900xtx-rocm-windows.md ；https://github.com/vrgamegirl19/comfyui-vrgamedevgirl/blob/main/Workflows/LTX-2_Workflows/Video_Builder/readme.md

### 4.2 常用视频工作流节点/插件清单

| 类别 | 节点包 | 用途 |
|---|---|---|
| 模型封装 | kijai 三件套：WanVideoWrapper / HunyuanVideoWrapper / CogVideoXWrapper，MochiWrapper，FramePackWrapper | 几乎所有主流开源视频模型的 ComfyUI 入口 |
| 官方 | ComfyUI-LTXVideo（Lightricks） | LTX-2/2.3/2.5 全功能 |
| 视频 IO | ComfyUI-VideoHelperSuite（VHS） | LoadVideo / VideoCombine / 抽帧 / 批量管理，视频工作流必备 |
| 量化加载 | ComfyUI-GGUF | 低显存跑大模型 |
| 控制 | ComfyUI-ControlNet-Aux + KSampler ControlNet | Canny/Depth/Pose 视频到视频 |
| 工具 | ComfyUI-KJNodes、comfyui_memory_cleanup | 尺寸/显存管理 |
| 社区增强 | 10S-Comfy-nodes（LTX 潜变量运镜/重定时）、WhatDreamsCost（LTX Director 2.0，1.7k stars） | 长片段拼接、镜头语言 |

来源：https://github.com/dan-ritual/video-ai-primer/blob/HEAD/06_COMFYUI_NODE_WORKFLOWS_GUIDE.md ；https://github.com/wildminder/awesome-ltx2/blob/HEAD/blocks/70_nodes.md ；https://github.com/alterpeace/runpod-comfy/blob/HEAD/docs/COMFYUI_WORKFLOW_STEERING.md

### 4.3 云托管 / Workflow-as-API（自建 GPU 的替代品）

| 方案 | 模式 | 要点 |
|---|---|---|
| Comfy Cloud（官方） | 订阅制，RTX 6000 Pro | 与本地同一 workflow 格式，API 调用；免费档不可调 API |
| RunComfy | 按量 GPU + 模型市场 | `runcomfy run <model>` CLI，Wan/Kling 等开箱即用 |
| ComfyDeploy | 开源，可自托管 | H100/A100/B200/H200/L40S…，workflow 版本管理、一键回滚 |
| RunPod Serverless | 按秒 GPU | ComfyUI-to-API 工具可自动打包 workflow 为 Docker 部署；约 $0.22/小时起（3090 社区云）|
| ComfyICU / RunningHub | 订阅/按量 | 前者 $10/月起 REST API；后者 9000+ 节点、40000+ 模型预装，按秒计费 |
| Replicate | 按次 | 开源视频模型最多的按量平台之一 |

来源：https://medium.com/@aguusstyu888/5-top-ways-to-deploy-comfyui-workflows-in-2026-bc8a5e07eff8 ；https://github.com/mohamedabdallah-14/prompt-to-asset/blob/HEAD/docs/research/20-open-source-repos-landscape/20d-comfyui-workflow-ecosystem.md ；https://github.com/slepetys-hue/2026_aec_cptx_demo_dml/blob/HEAD/deployment/source/profiles/aec-cptx/skills/creative/comfyui/SKILL.md

---

## 5. 已验证信息 vs 待验证/传闻

### ✅ 已验证（多独立来源交叉，2026 年内）

1. Sora 消费端 2026-04-26 关闭、Videos API 2026-09-24 关闭且无替代——OpenAI 官方退役表 + 多家媒体交叉确认。
2. Veo 3.1 价格体系（$0.40/s 标准、$0.15/s Fast、$0.03–0.05/s Lite）——Google 定价文档 + 第三方实测 + fal 牌价三方一致。
3. MiniMax H3 于 2026-07-31 发布、08-03 开源 Base 权重、33B、原生立体声音频——MiniMax 官方博客 + 路透 + 社区技术拆解一致。
4. Seedance 2.5（2026-06-23）原生 30 秒 + 50 多模态参考——字节 FORCE 大会 + 火山方舟文档转述 + 媒体一致。
5. Runway API $0.12/s（Gen-4.5）、Turbo $0.05/s——Runway 官方定价指南口径。
6. LTX-2 系列已进 ComfyUI 核心、kijai wrapper 为事实标准——ComfyUI 官方仓库 + 社区文档一致。

### ⚠️ 待验证 / 传闻（单来源或编辑性推断，引用时请标注）

1. **Veo 3.1 GA 版退役日期 2026-11-17**——出自 mediovsky 一文引用的 Google 列表，尚未见 Google 官方退役表原文；若属实，Veo 选型需做版本迁移预案。
2. **"Google 免费额度公告 24 小时后 Sora 关闭"是竞争所致**——frontiernews.ai 的编辑性解读，OpenAI 官方口径是既定的两阶段关停计划。
3. **Luma "Ray3.14"**——仅一篇第三方评测提及（haxzie/genmotion），Luma 官方口径为 Ray 3.2。
4. **Seedance 2.5 满血版"1000 万元起年框"门槛**——单篇中文知识库转述，未见官方确认。
5. **fal.ai "视频 API 市占率 44%"**——出自单个开发者集成文档的陈述，无第三方审计。
6. **Pika 免费档水印/商用政策**——不同来源说法矛盾（有称无水印、有称有水印），以 pika.art 实时页面为准。
7. **Kling 官方定价页当前报价**——曾被报告无法读取，第三方聚合价不能替代官方报价。
8. **"AI 短剧市场见顶、Seedance 降价"**——钛媒体 2026-09-16 单篇渠道报道，属行业信源转述，趋势可信度较高但数字（5.5 折/8.8 折）为渠道商单方说法。

---

## 6. 对 video-factory-lab 流水线设计的直接建议

> 目标场景：AI 漫剧/短剧（长叙事、角色一致性、批量镜头）+ 产品营销视频（高质感、短平快）。

**建议 1：采用"混合架构"，按镜头价值分流——不要押注单一路线。**
- **关键镜头**（主角特写、仿真人表演、30 秒一镜到底）：商用 API，首选 **Seedance 2.5**（原生 30s + 50 参考，短剧拼接成本减半）或 **Veo 3.1 Fast**（对白/音效原生，$0.15/s）。
- **量大管饱的过渡镜头/B-roll/营销素材批量版**：开源自部署或云 GPU——**Wan 2.7/3.0**（Apache 2.0，无商用限制）或 **LTX-2.3 蒸馏版**（8 步草稿极快），单条成本可压到 $0.02–0.05。
- **依据**：Seedance/H3 的 2026-09 价格战证明"不是所有镜头都值得调用最贵的模型"已成行业共识（https://www.21jingji.com/article/20260916/herald/8f990c6444eb701692731b4e72375e0d.html）。

**建议 2：API 接入层以 fal.ai 为默认网关，并从 Day 1 做双供应商 fallback。**
- fal 统一 schema、无月费，Kling/Veo 官方同价，queue+webhook+失败不计费，新增模型多为 day-0 上线；量起来后 Seedance 切 **BytePlus ModelArk 直连**（约半价）。
- Sora 的教训：任何模型都可能在 6 个月通知后消失。接入层必须做"模型别名 → 可替换实现"的抽象，并对 Veo 3.1 这类已传出退役日期的模型做版本监控。
- 来源：https://github.com/belcort-sdn-bhd/fikirtive/blob/HEAD/docs/research/genapi-provider-research.md ；https://mediovsky.com/sora-api-shutdown/

**建议 3：音频策略单独设计——"原生音频"不是免费午餐。**
- 需要口型/对白的漫剧镜头：优先 **Wan 2.7 音频驱动模式**（TTS 先行、`audio_url` 锁口型，音画解耦、可控静音）或 Veo 3.1/H3 原生音频。
- 实测教训：Veo 3.1 **不响应 silence 指令**（prompt 无法让它静音），且会把视频生成和音频合成绑死，导致下游无法独立控制语音节奏。凡对配音有精确要求的管线，音频必须走"TTS 先行 + 口型同步"路线，而非依赖原生音频。
- 来源：https://github.com/bthurlow/artificer-mcp/blob/HEAD/docs/plans/fal-ai-integration.md

**建议 4：ComfyUI 定位为"研发工作台"，而非生产执行器。**
- 本地/云端 ComfyUI（kijai wrappers + LTX 官方节点 + VHS）用于工作流原型、LoRA 风格调试、镜头语言实验；其 workflow JSON 可直接导出。
- 生产执行走 **Serverless API**（RunPod / ComfyDeploy / RunComfy）或 fal/Replicate 按量调用，避免自建 GPU 集群的运维负担；ComfyUI workflow 与云 API 同格式，迁移成本低。
- 来源：https://github.com/dan-ritual/video-ai-primer/blob/HEAD/06_COMFYUI_NODE_WORKFLOWS_GUIDE.md ；https://medium.com/@aguusstyu888/5-top-ways-to-deploy-comfyui-workflows-in-2026-bc8a5e07eff8

**建议 5：成本核算用"每采用秒"，并给每个模型设"采用率"指标。**
- 核算口径：`每采用秒成本 =（生成+重试+修正+剪辑+音频+审核）/ 实际采用秒数`。便宜单价 × 低采用率可能贵过高单价 × 高采用率（实测算例：Kling $0.84 vs Veo $0.80 每采用秒，几乎打平）。
- 落到流水线：在任务层记录每个模型/提示词模板的"一次过审率"，每月复核一次模型选型——2026 年的价格战意味着最优解每季度都在变。
- 来源：https://github.com/rrrrrredy/research-toolkit/blob/HEAD/evals/diagnostics/2026-09-07/reports/video-production/reviews/before-first-source-review.md

---

## 附录：主要来源（按主题）

**市场格局与关停事件**
- https://magichour.ai/blog/what-is-sora
- https://mediovsky.com/sora-api-shutdown/
- https://tech-insider.org/veo-3-1-api-setup-sora-2-migration-2026/
- https://www.knowmouth.com/openai-sora-shutdown-sora-2-api-discontinued
- https://www.technology.org/2026/09/07/ai-videos-new-power-trio-seedance-2-5-minimax-h3-wan-3/
- https://news.marsbit.co/20260923163210637160.html
- https://induwara.lk/tools/ai-video-generator-comparison

**价格与 API（含聚合器）**
- https://github.com/juspay/director/blob/HEAD/video-production/library/docs/VIDEO-GEN-LANDSCAPE-2026Q1.md
- https://github.com/juspay/director/blob/HEAD/video-production/library/docs/VIDEO-API-RESEARCH.md
- https://github.com/anil-matcha/awesome-ai-video-models
- https://github.com/localsymmetry/lofn/blob/HEAD/output/analysis/2026-08-15_video-model-landscape.md
- https://github.com/madponyinteractive/cubric-vision/blob/HEAD/docs/proprietary-models-research/01-aggregators.md
- https://github.com/madponyinteractive/cubric-vision/blob/HEAD/docs/proprietary-models-research/01b-more-aggregators.md
- https://github.com/bthurlow/artificer-mcp/blob/HEAD/docs/plans/fal-ai-integration.md
- https://github.com/haxzie/genmotion/blob/HEAD/apps/web/content/blog/best-ai-video-generation-models.md
- https://github.com/belcort-sdn-bhd/fikirtive/blob/HEAD/docs/research/genapi-provider-research.md
- https://github.com/rrrrrredy/research-toolkit/blob/HEAD/evals/diagnostics/2026-09-07/reports/video-production/reviews/before-first-source-review.md
- https://lifestyle.filmtelevisionauditions.com/story/337595/top-5-fastest-ai-video-generators-in-2026/

**中国模型（可灵/Seedance/即梦/Vidu/Hailuo）**
- https://github.com/ethanniworld/ai-playbook-2026/blob/HEAD/knowledge/bytedance/seedance-series.md
- https://github.com/moemu/richidrama/blob/HEAD/docs/plans/2026-08-10-volcengine-official-pricebook.md
- https://www.21jingji.com/article/20260916/herald/8f990c6444eb701692731b4e72375e0d.html
- https://www.eet-china.com/mp/a503020.html
- https://github.com/ht-shaipe/soma/blob/HEAD/docs/china-ai-video-api-research.md
- https://www.fahimai.com/zh/kling-ai
- http://www.360doc.com/content/26/0216/00/57798620_1170117513.shtml
- https://www.stnn.cc/c/2026-04-10/4055997.shtml
- https://github.com/gvclab/video-generation-101/blob/HEAD/sources/research_20260829_minimax_h3.md
- https://github.com/anastasiyaw/knowledge-space/blob/HEAD/docs/projects/minimax-h3.md

**厂商定价页解读（Luma/Pika/Runway/Veo）**
- https://www.eesel.ai/blog/luma-ai-pricing
- https://magichour.ai/blog/luma-dream-machine-pricing
- https://magichour.ai/blog/pika-labs-pricing
- https://www.therundown.ai/tools/pika
- https://www.therundown.ai/tools/aleph
- https://www.aixploria.com/en/runway-gen-4-5/
- https://github.com/heygen-dev/runway-gen-4/blob/HEAD/README.md
- https://vidofy.ai/en/models/runway/gen-4-aleph
- https://github.com/kacky000/aitoolpick/blob/HEAD/src/content/blog/google-veo-pricing-2026.md
- https://github.com/mua47105-hue/agentic-ai-video-knowledgebase/blob/HEAD/kb/wiki/archive/google-veo-3.md
- https://github.com/mua47105-hue/agentic-ai-video-knowledgebase/blob/HEAD/kb/wiki/archive/pika-2-5.md
- https://www.veo3ai.io/blog/veo-3-api-integration-guide-2026
- https://github.com/boreilly-dc/sota-reference/blob/HEAD/media-generation/video-generation-ai.md

**开源模型**
- https://www.thundercompute.com/blog/best-open-source-ai-video-generation-models
- https://medium.com/@jon_davis/the-open-source-ai-video-stack-in-2026-what-actually-works-9b20fb281ec2
- https://github.com/seanwinslow28/anima/blob/HEAD/docs/open-sourced-video-model-research/Open-Source-Video-Models-for-ComfyUI-Gemini.md
- https://github.com/ishandutta2007/video-generation-landscape/blob/HEAD/README.md
- https://github.com/quriosity-agent/articles/blob/HEAD/2026-03-18/chris-defi-2033832631083966913-analysis-en.md
- https://github.com/opensource-works/awesome-minimax-h3-prompts/blob/HEAD/README.md
- https://github.com/cobdog/minimax-desktop/blob/HEAD/docs/research/ecosystem-2026-09.md
- https://vercel.com/changelog/wan-3-0-now-available-on-ai-gateway
- https://github.com/civitai/civitai-developer-docs/blob/HEAD/orchestration/recipes/wan.md
- https://github.com/atlascloudai/awesome-wan-3.0-prompts

**ComfyUI 生态**
- https://github.com/Lightricks/ComfyUI-LTXVideo/
- https://github.com/dan-ritual/video-ai-primer/blob/HEAD/06_COMFYUI_NODE_WORKFLOWS_GUIDE.md
- https://github.com/wildminder/awesome-ltx2/blob/HEAD/blocks/70_nodes.md
- https://github.com/joshwaamein/joshwaamein.github.io/blob/HEAD/_posts/2026-04-05-comfyui-amd-rx7900xtx-rocm-windows.md
- https://github.com/alterpeace/runpod-comfy/blob/HEAD/docs/COMFYUI_WORKFLOW_STEERING.md
- https://github.com/artokun/comfyui-mcp/blob/HEAD/docs/blog/ltx-2.3-comfyui.mdx
- https://github.com/vrgamegirl19/comfyui-vrgamedevgirl/blob/main/Workflows/LTX-2_Workflows/Video_Builder/readme.md
- https://github.com/septa-serpenta-seraph/vesper/blob/HEAD/skills/.archive/ltx23-comfyui-setup/SKILL.md
- https://medium.com/@aguusstyu888/5-top-ways-to-deploy-comfyui-workflows-in-2026-bc8a5e07eff8
- https://github.com/mohamedabdallah-14/prompt-to-asset/blob/HEAD/docs/research/20-open-source-repos-landscape/20d-comfyui-workflow-ecosystem.md
- https://github.com/slepetys-hue/2026_aec_cptx_demo_dml/blob/HEAD/deployment/source/profiles/aec-cptx/skills/creative/comfyui/SKILL.md
- https://github.com/sutchan/agent-skills-hub/blob/HEAD/skills/ai-video-generation-runcomfy/SKILL.md
- https://www.openpr.com/news/4238532/runninghub-unleashes-aigc-creation-major-price-cuts-slash
