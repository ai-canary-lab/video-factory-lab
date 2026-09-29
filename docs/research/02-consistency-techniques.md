# 专题 2：角色 / 场景 / 风格一致性技术调研

> 调研日期：2026-09-29
> 报告人：子调研员（专题 2）
> 目标项目：video-factory-lab（AI 漫剧 / 短剧 + 产品营销视频流水线）

---

## 0. 信息时效说明

- 本报告以 2026 年资料为主（多数来源为 2026-03 ~ 2026-09 更新）。
- ⚠️ 约 2025 年及更早的资料（DreamBooth 论文 2023、MotionCtrl/CameraCtrl 2024、SDXL 时代的 AnimateDiff/IP-Adapter 套路）在"时效"栏中单独标注，过时结论已剔除或降级。
- 每个关键结论后附来源链接（URL 原样引用）。未直接抓取的二级来源用「转引」标注。

---

## 1. 总览：一致性问题的本质与 2026 年主流答案

扩散模型每次从随机噪声起步去噪，"三十多岁蓄胡男子"这类文字提示对应训练数据里数百万张脸，输出是"平均脸"而非某个人；复用同一提示词也会因起点不同而漂移。镜头越多，漂移累积越严重。

**2026 年社区共识工作流**（转引自中文一致性手册，2026-08-18 审核）：先在单张参考图中锁定身份，每镜复用该参考——比纯提示词更一致、更精确、更可靠；整片级稳定则上 LoRA。

来源：https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/39-image-consistency/character-consistency.md

---

## 2. 参考图（Reference Image）驱动的一致性方案 ✅ 已验证

### 2.1 五种方法的社区横评（2026-08，中文）

| 方法 | 原理 | 适合场景 | 一致性 |
|---|---|---|---|
| Character Reference（img2img） | 上传参考图，模型新场景匹配身份 | 几乎所有人，最快路径 | 高 |
| 转面图 Turnaround Sheet | 正/侧/背/3/4 多角度参考 | 给图生视频提供角度与运动上下文 | 高 |
| 提示词复用 | 同 seed + 逐字角色描述跨代复用 | 单一外观快速变体 | 中 |
| LoRA 训练 | 15–50 张图训练小型定制模型 | 数百镜头、近零漂移 | 极高 |
| 换脸 Face Swap | 先生成后逐帧换脸 | 事后修复 | 高（仅脸） |

选择逻辑：零学习成本 → Character Reference；跨镜头多角度 → 转面图；整片级稳定 → LoRA；已成片要修 → Face Swap。

来源：同上 https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/39-image-consistency/character-consistency.md

### 2.2 各主流工具 2026 年的参考机制（已验证）

- **Kling 3.0（2026-02）**：Omni Reference + Elements，单个 Element 最多挂 7 张角色参考，每镜 @ 提及；多角色 coreference（3+ 角色互不串脸）；O3 支持声音绑定。来源：同上手册 + https://github.com/yyforever/aividpipeline-skills（fal.ai 流水线文档，2026 年初更新）
- **Veo 3 / 3.1**：Ingredients to Video 参考系统，"喂参考图锁身份 + 每镜用完全相同的角色描述"；原生同步音频+口型。来源：同上手册
- **Seedance 2.0**：Reference to Video + 提及系统，`@Image1` 指认首帧、`@Image2-4` 指认角色外观、`@Audio1` 指认对白，提示词只做"指派"，信息尽量放在参考图里。来源：同上手册
- **Midjourney**：`--oref`（Omni Reference）+ `--ow` 权重；改角色细节时降低权重；会软化雀斑/纹身等细节。来源：同上手册
- **ComfyUI 本地栈**：PuLID + InstantID + IP-Adapter + LoRA 叠加，最高控制精度。来源：同上手册

### 2.3 铁律与雷区（已验证，多源交叉）

- **铁律：先在静止图像里锚定身份，再动画化它。** 端到端管线 = 设计角色 → 建参考集（转面图正/侧/背/3/4 + 服装/道具/表情）→ 每镜挂角色参考 → 锁定图转视频 → 剪辑调色。来源：https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/39-image-consistency/character-consistency.md
- **四大崩点**：帧间漂移（转头再转回脸变形）、侧面/背面崩坏（参考系统多训练于正面）、服装/道具漂移（视频模型优先保脸）、链式拼接的漂移跳变。来源：同上
- **常见错误**：纯文字描述跨镜（必然漂移）、只锁正脸、每镜微调角色描述、服装不锁、参考图糊/暗/非正面（参考质量决定一致性上限）。来源：同上

### 2.4 ComfyUI 叠加栈（已验证，Apatero 2026）

分层叠加，每层管一个维度，声称一致性 ~95%：

```
Layer 1 角色 LoRA（身份嵌入，20 张图 / rank 8，触发词如 elena_ohwx）
Layer 2 IP-Adapter FaceID（面部强化，参考正面证件照，权重 0.8）
Layer 3 ControlNet OpenPose（姿态锁定）
Layer 4 事后 inpainting（修眼、手、服装细节）
```

来源：https://github.com/0xzgbot/forge-nps-v01/blob/HEAD/hermes_home/skills/skill_character_consistency/SKILL.md；转引 Apatero：https://apatero.com/blog/comfyui-character-consistency-advanced-workflows-2026

---

## 3. LoRA / DreamBooth 轻量微调在视频人物一致性上的应用 ✅ 已验证（金标准）

### 3.1 LoRA：整片级稳定的实用默认

- **训练配方**：rank 16–32、10–30（至多 50）张图、稀有触发词（`ohwx`/`sks`/`zkz`）、标注"描述除身份外的一切"让身份坍缩到触发词上、prior-preservation。磁盘仅 ~40MB，可叠加。来源：https://github.com/mohamedabdallah-14/prompt-to-asset/blob/HEAD/docs/research/15-style-consistency-brand/15a-consistent-character-and-mascot.md
- **面部 LoRA 实战**：Character Sheet 做统一且有变化的数据集，在 RAW 底模上 3000–5000 步收敛；测试要覆盖近景与远景（拒绝"近景稳、远景崩"）。来源：https://www.youtube.com/watch?v=A32qmJNbl4Q（Krea2 LoRA 中文教程，2026）
- **训练到视频闭环**：从视频抽帧 → 去重（phash/SSIM）→ WD14/Florence-2 打标 → kohya_ss 训练 → AnimateDiff 运动模块 + 角色 LoRA 生成视频 → ToonCrafter 补帧。来源：https://github.com/acfharbinger/image-toolkit（roadmap，2026-09 更新）
- **2026-05 评估结论**：FLUX.1-dev 上用 30 张关键帧训练角色 LoRA（rank 16–32，1500–3000 步）是"身份锁定的金标准"，可跨数百帧锚定身份。来源：https://github.com/seanwinslow28/anima/blob/HEAD/docs/Image-Model-DR-2026/prompt-2-perplexity.md

### 3.2 训练无关（training-free）的人脸锁定 ✅ 已验证

- **PuLID / PuLID-FLUX**：无需训练的面部锁定；非写实风格有"贴脸感"风险，缓解办法是 `timestep_to_start_inserting_ID` 设为 0–1 并与角色 LoRA 以 ≤0.6 强度叠加。来源：同上 anima 文档
- **InstantCharacter（腾讯/HunyuanDiT，Apache 2.0）**：截至 2026 年中"最强的免训练适配器"，跑在 FLUX.1-dev 上，24GB 显存可跑，跨二次元与手绘风格兼容性好。来源：同上 anima 文档
- **InstantID（2024-01）**：用人脸识别模型提取身份、经 cross-attention 注入，免调优。来源：https://github.com/seanwinslow28/anima/blob/HEAD/docs/Image-Model-DR-2026/prompt-1-chatgpt.md（转引其 2026-09 更新的研究笔记）

### 3.3 DreamBooth：地位下降 ⚠️ 时效标注

- DreamBooth 的 prior-preservation loss（稀有 token + 类别正则图）仍是防止"概念坍缩"的理论基础，但 2026 年社区实际**优先用 LoRA**（体积、组合性、生态支持全面占优）；完整 DreamBooth 微调只在需要极致保真时考虑，成本高（数百张图 / 天级训练，4090）。来源：https://github.com/mohamedabdallah-14/prompt-to-asset/blob/HEAD/docs/research/15-style-consistency-brand/15a-consistent-character-and-mascot.md；https://github.com/seanwinslow28/anima/blob/HEAD/docs/Image-Model-DR-2026/prompt-1-chatgpt.md
- 视频基座上的 LoRA 训练支持仍偏实验性：lora-pilot（2026-09 更新）标注 HunyuanVideo / Cosmos / Wan2.1-2.2 的 LoRA 训练为 experimental，DreamBooth 多为不支持或 limited。来源：https://github.com/vavo/lora-pilot/blob/HEAD/docs/reference/supported-models.md

### 3.4 视频侧的身份锁定案例 ✅ 已验证

- **AnimeLoom（开源，2026-03 更新）**：SDXL + LoRA + IP-Adapter 生成身份锁定关键帧 → Wan2.2-I2V-Animate 驱动动画 → Wan-Animate face lock（驱动片段给动作、关键帧给人脸参考）；身份一致性从 v1.0 的 ~70% 演进到 v2.0 的 **~95%+**。来源：https://github.com/caiyilian/novel-voice-cast/blob/HEAD/docs/开源动画视频生成项目调研.md（调研日期 2026-06-30）

---

## 4. 首尾帧控制（FLF2V / Keyframe Conditioning）✅ 已验证（2026 年最实用的运镜+一致性手段）

### 4.1 能力分布实测（2026-09，29 个模型的 API 级核查）

- OpenRouter 公开列表 29 个视频模型中，**15 个接受尾帧**；字段名不统一，有 4 种约定：`first_frame_url`/`last_frame_url`（Wan 2.2 kf2v）、`media: [{type: first_frame}...]`（Wan 2.7）、`start_image`/`end_image`（Kling/Luma/Vidu）、`image`/`last_frame_image`（Seedance/PixVerse/LTX）、`image`/`lastFrame`（Veo 3.1）。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/
- **已确认仅首帧**（传两帧会被静默忽略、照单收费）：`alibaba/wan-3`、`runwayml/gen-4.5`、`minimax/hailuo-2.3`、`kwaivgi/kling-v2.6`、`wan-video/wan-2.5-i2v-fast`。来源：同上
- 开源侧：**Wan 2.2 I2V-A14B 原生支持 FLF2V**（`WanFirstLastFrameToVideo` 节点，2025-08 起），720p×5s@16fps；**LTX-Video 0.9.5+** 的 `LTXVKeyFrameConditioning`、LTX 2.x 支持任意 latent 帧条件（首/中/尾任意组合）。来源：https://github.com/seanwinslow28/anima/blob/HEAD/docs/open-sourced-video-model-research/compass_artifact_wf-68c440a4-26a1-4188-9a4a-89bb65d894a9_text_markdown.md；https://github.com/vincentgourbin/ltx-video-swift-mlx/blob/HEAD/README.md；https://github.com/dkackman/diffusers-workflow/blob/HEAD/docs/proposals/audits/2026-09-07-ltx-2.5-audit.md

### 4.2 关键帧 trade-off（实测数据，已验证）

- **锚定两端会压制运动**：首=尾的循环测试中，关键帧模型 churn 约为纯 I2V 的 **1/10**（树叶 2.67 vs 34.14）。要循环不如"正常生成 + 本地 crossfade/ping-pong"，运动量大且无缝。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/
- **强度参数**：LTX 文档建议首尾用 *guiding* 而非 *replacing*（强度略低于 1.0，如 0.95），插值更平滑。来源：https://github.com/dkackman/diffusers-workflow/blob/HEAD/docs/proposals/audits/2026-09-07-ltx-2.5-audit.md
- **提示词无法增加运动量**：实测"点名每个运动物体"的提示词对 churn 无系统性提升，限制因素是两端锚定本身；要更多运动就加长片段或去掉尾帧。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/

---

## 5. 运镜控制（Camera Motion Control）⚠️ 已验证但分层

### 5.1 2026 年可用的方案（已验证）

- **Wan 2.2 Fun Camera**：ComfyUI 最易入口，`WanCameraEmbedding` 节点选 "Zoom In / Pan Left / Zoom Out" 等预设运镜，HIGH noise 35–40 步 + LOW noise 10–15 步两阶段采样。来源：https://github.com/ismael-joffroy-chandoutis/open-source-cinema/blob/HEAD/Generative-Pipeline-March-2026.md（2026-03 更新，转引 ComfyUI 官方教程）
- **ReCamMaster**：给已有视频换一条新相机轨迹重拍（Wan2.1 1.3B 权重，ComfyUI-WanVideoWrapper 集成）。来源：同上
- **Scope（腾讯）**：基于 Wan 2.2 的路径指定运镜（pullback、dolly、crane、orbit），单图 + 相机路径输入，Apache 2.0 开源。来源：https://github.com/minecraft9101010/ai-tools-mcp/blob/HEAD/cards/scope-camera-control.md（2026-08-16 审核）
- **Wan VACE**：唯一真正 mask-native 的系列（inpainting/outpainting + 首尾帧），fal 端点按秒计费；注意 VAE 编解码往返会轻微改变"未动"像素。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/

### 5.2 实验 / 学术阶段（待验证）

- **CamC2V**（3DV 2026，波恩大学）：context-aware 可控视频生成，指标优于 MotionCtrl/CameraCtrl，但 repo 仅 14 star，属研究代码。来源：https://github.com/LDenninger/CamC2V
- **GEN3C**（NVIDIA）：3D cache-informed 相机控制，几何精度最高，但部署重。来源：转引 https://github.com/ismael-joffroy-chandoutis/open-source-cinema/blob/HEAD/Generative-Pipeline-March-2026.md
- **TrajectoryCrafter**（ICCV 2025 Oral）：单目视频 6DoF 重定向，需 28GB 显存。来源：同上
- ⚠️ **MotionCtrl / CameraCtrl（2024）**：U-Net 时代的相机姿态控制，已被 DiT 范式边缘化；CamC2V 论文的对比表显示其 TransErr/RotErr 落后新方法，仅作历史参考。来源：http://arxiv.org/pdf/2404.02101v1；https://github.com/LDenninger/CamC2V

### 5.3 一个坑（已验证）

"Motion brush 画哪里动哪里"在 Web App 常见，但 **API 侧几乎没有**；Kling 在 Replicate/fal 上的 motion-control 模型实际是"参考视频动作迁移"（`image_url` + `video_url`），不是画笔。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/

---

## 6. 多镜头 / 多分镜之间保持一致的方法 ✅ 已验证

### 6.1 主流范式：分镜图 → 图生视频（storyboard-first）

社区反复验证的最稳范式（不是文本直出视频）：

1. **角色表**：用图像模型锁定角色一次（character sheet）。
2. **分镜静帧**：每个场景生成一张关键帧静帧（可基于角色表编辑，保证身份）。
3. **动画化**：每张关键帧做 image-to-video。
4. **装配**：剪辑成片。

> "Lock the composition as a still you can review, then animate it. You get far more control over framing, character, and continuity than letting the video model invent each shot from text."

来源：https://github.com/vericontext/vibeframe/blob/HEAD/docs/ai-video-prompting.md（转引 seedance.tv 2026 一致性指南、deepfiction.ai 2026 管线文章）

### 6.2 商业模型的原生多镜头能力（2026，已验证）

- **Kling 3.0（2026-02）**：multi-shot storyboard，一次最多 6 个镜头、每镜独立 prompt/时长、总计 15s；Element 角色参考跨镜；多角色 coreference。来源：https://github.com/yyforever/aividpipeline-skills（2026 年初文档）
- **Seedance 2.0**：多模态引用（≤9 图 + ≤3 视频 + ≤3 音频），`@Image/@Video/@Audio` 跨镜指派；社区已有 Vidtory Drama Studio 等一键短剧工具（6 层身份锚定 + Character Bible）。来源：https://github.com/tuha1994/vidtory-seedance-2.0-drama-studio
- **Veo 3.1**：Ingredients 参考 + 每镜逐字相同的角色描述。来源：dsh-directorx 手册（见 2.2）

### 6.3 开源 / 本地的多镜头衔接手法（已验证）

- **尾帧链式拼接**：clip N 的尾帧 = clip N+1 的首帧；短片段 5–8 秒一段；每一跳都是新的漂移机会，需配合参考图/LoRA 压住。来源：https://github.com/0xzgbot/forge-nps-v01/blob/HEAD/hermes_home/skills/skill_character_consistency/SKILL.md
- **单基底 plate**：先生成一张主场景静帧（master location still），各角度用 inpaint/reframe 编辑而非重新生成，保证光照与环境连续。来源：https://github.com/hoodini/ai-agents-skills/blob/HEAD/skills/director/references/17-ai-storyboard-prompting-and-keyframes.md
- **统一风格码**：全片共用一个 `--sref` / 风格参考图锁定色彩科学与美术。来源：同上
- **三路参考分离**（Flick 片厂实践，2026）：构图草图（唯一构图参考）+ 侧脸参考（唯一脸/发型参考）+ 全身参考（唯一服装参考）三路同时输入，提示词逐路声明"100% 匹配"，换来构图与身份双重精确。来源：dsh-directorx 手册（见 2.1）

---

## 7. 推荐优先级排序（2026-09-29 视角）

### Tier 0 — 已被大量案例验证，闭眼用

1. **参考图 + 转面图前置锁定**（"先在静止图像里锚定身份，再动画化它"）——所有工具链通用，零训练成本，中文短剧社区（SegmentFault 保姆级实践）与海外片厂双重验证。来源：dsh-directorx 手册；https://github.com/vericontext/vibeframe/blob/HEAD/docs/ai-video-prompting.md
2. **分镜图 → 图生视频（storyboard-first）**，每镜逐字复用角色描述 + 挂参考——Seedance/Kling/Veo 官方指南与社区工具（Vidtory、VibeFrame）一致推荐。来源：同上
3. **FLF2V 首尾帧锚定**（Wan 2.2 原生 / LTX 关键帧条件）——2026-09 实测 15/29 模型支持，开源侧 Wan 2.2 是唯一原生 FLF2V 的开放权重。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/；anima 开源模型横评
4. **尾帧链式拼接**（5–8s 短片段 + 尾帧作下一段首帧）——长镜头标准做法。来源：skill_character_consistency

### Tier 1 — 已验证有效，需要投入训练/工程

5. **角色 LoRA**（rank 16–32，15–50 张图，稀有触发词）——整片级"金标准"，与 IP-Adapter/PuLID/ControlNet 叠加达 ~95% 一致性（Apatero）。来源：anima 2026-05 评估；https://github.com/0xzgbot/forge-nps-v01/blob/HEAD/hermes_home/skills/skill_character_consistency/SKILL.md
6. **PuLID / InstantID / InstantCharacter**（免训练面部锁定）——InstantCharacter（腾讯，Apache 2.0）是 2026 年中免训练最强项。来源：https://github.com/seanwinslow28/anima/blob/HEAD/docs/Image-Model-DR-2026/prompt-2-perplexity.md
7. **Wan 2.2 Fun Camera / ReCamMaster**（运镜控制）——ComfyUI 可直接用的预设运镜与重拍。来源：https://github.com/ismael-joffroy-chandoutis/open-source-cinema/blob/HEAD/Generative-Pipeline-March-2026.md
8. **AnimeLoom 式 face lock**（Wan-Animate：关键帧给脸、驱动片段给动作，~95%+）——小说→动画流水线的现成参考实现。来源：https://github.com/caiyilian/novel-voice-cast/blob/HEAD/docs/开源动画视频生成项目调研.md

### Tier 2 — 实验阶段，观望或小规模试

9. **视频基座 LoRA 微调**（Wan 2.x / HunyuanVideo / Cosmos）——lora-pilot 2026-09 仍标 experimental。来源：https://github.com/vavo/lora-pilot/blob/HEAD/docs/reference/supported-models.md
10. **完整 DreamBooth 视频微调**——成本高、生态弱，已被 LoRA 替代大半。来源：anima 研究笔记
11. **CamC2V / GEN3C / TrajectoryCrafter / Scope**——学术或刚开源，部署重、star 少，待社区验证。来源：见 5.2
12. **LTX Cinemagraph LoRA**（Lightricks 官方循环 cinemagraph）——仅 ComfyUI，无托管端点，场景窄。来源：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/

---

## 8. 对 video-factory-lab 流水线设计的直接建议

1. **角色卡 = "参考图包 + LoRA + DNA 文本"三件套，版本化管理**。参考图包：转面图（正/侧/背/3/4）+ 服装 anchor pack（每套服装 5+ 张多角度）+ 表情参考；DNA 文本：逐字锁定的角色描述（跨镜一字不改）；条件允许时再训一个 rank 16–32 的角色 LoRA（15–50 张图）。这是 2026 年验证度最高的分层方案（叠加栈 ~95%）。
2. **场景卡独立于角色卡**：每个场景一张 master location still 作为"基底 plate"，各镜头用 inpaint/reframe 编辑而非重生成；场景参考图作为与角色参考并列的第二参考通道；全片共用一个风格参考（sref）锁色彩美术。
3. **一致性检查点放在三个位置**：① 分镜静帧评审（身份/服装/场景三路对照角色卡、场景卡——这是最便宜的拦截点）；② 每段视频生成后做"首帧 vs 参考图"人脸相似度抽检（ArcFace 向量距离）；③ 链式拼接处检查尾帧→首帧的漂移跳变。不要等到成片才发现换脸。
4. **默认走 storyboard-first + FLF2V**：每镜先出锁定静帧 → 首尾帧锚定转视频（Wan 2.2 原生 FLF2V / LTX 关键帧条件）→ 5–8 秒短片段尾帧链式拼接。注意实测结论：首尾双锚会压制运动量约 10 倍，大动作镜头去掉尾帧或加长片段。
5. **运镜用预设而非玄学提示词**：Wan 2.2 Fun Camera 的预设运镜（push/pull/pan）+ ReCamMaster 重拍，比"camera slowly dolly in"这类提示词可复现得多；motion brush 的 API 缺失是已知坑，不要在设计里依赖它。

---

## 附：主要来源清单（按抓取时间 2026-09-29）

- 中文一致性手册（2026-08-18 审核）：https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/39-image-consistency/character-consistency.md
- 首尾帧模型实测（2026-09）：https://mer.vin/2026/09/which-ai-video-models-actually-take-a-first-and-last-frame/
- ComfyUI 叠加栈：https://github.com/0xzgbot/forge-nps-v01/blob/HEAD/hermes_home/skills/skill_character_consistency/SKILL.md
- LoRA/角色训练 2026-05 评估：https://github.com/seanwinslow28/anima/blob/HEAD/docs/Image-Model-DR-2026/prompt-2-perplexity.md
- 开源视频模型横评（含 FLF2V，2026-04）：https://github.com/seanwinslow28/anima/blob/HEAD/docs/open-sourced-video-model-research/compass_artifact_wf-68c440a4-26a1-4188-9a4a-89bb65d894a9_text_markdown.md
- LoRA vs DreamBooth 对比：https://github.com/mohamedabdallah-14/prompt-to-asset/blob/HEAD/docs/research/15-style-consistency-brand/15a-consistent-character-and-mascot.md
- LoRA 训练工艺：https://github.com/ryannel/skills/blob/HEAD/skills/generative-media/character-lora-training/SKILL.md
- LTX 关键帧条件：https://github.com/vincentgourbin/ltx-video-swift-mlx/blob/HEAD/README.md；https://github.com/dkackman/diffusers-workflow/blob/HEAD/docs/proposals/audits/2026-09-07-ltx-2.5-audit.md
- 运镜/重拍（2026-03）：https://github.com/ismael-joffroy-chandoutis/open-source-cinema/blob/HEAD/Generative-Pipeline-March-2026.md
- Scope 相机控制（2026-08）：https://github.com/minecraft9101010/ai-tools-mcp/blob/HEAD/cards/scope-camera-control.md
- AnimeLoom 流水线（调研 2026-06-30）：https://github.com/caiyilian/novel-voice-cast/blob/HEAD/docs/开源动画视频生成项目调研.md
- 分镜优先范式：https://github.com/vericontext/vibeframe/blob/HEAD/docs/ai-video-prompting.md
- 分镜一致性栈：https://github.com/hoodini/ai-agents-skills/blob/HEAD/skills/director/references/17-ai-storyboard-prompting-and-keyframes.md
- 视频基座 LoRA 支持度（2026-09）：https://github.com/vavo/lora-pilot/blob/HEAD/docs/reference/supported-models.md
- Kling 3.0 多镜头（2026-02）：https://github.com/yyforever/aividpipeline-skills
- Vidtory 短剧工具：https://github.com/tuha1994/vidtory-seedance-2.0-drama-studio
- Krea2 LoRA 中文教程（2026）：https://www.youtube.com/watch?v=A32qmJNbl4Q
- ⚠️ 时效较旧、仅作历史参考：Wan2.2-Animate CSDN 实测（约 2025 年底）：https://blog.csdn.net/gogoMark/article/details/153318600；MotionCtrl/CameraCtrl 论文（2024）：http://arxiv.org/pdf/2404.02101v1
