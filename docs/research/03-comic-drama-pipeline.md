# 专题3：AI 漫剧/短剧生产流水线调研报告

- **调研日期**：2026-09-29
- **调研方式**：公开网络资料（中英文），browser.search / browser.open 文本抓取
- **信息时效说明**：优先采用 2026 年资料；文中标注每条关键数据的来源日期。2026 年是 AI 漫剧"商业化兑现年"，半年前的数据都可能过时，请注意。

> 术语口径：AI 漫剧 = 以 AI 生成的静态漫画风格画面 + 配音剪辑为主的剧集（重资产在图，不在视频）；AI 短剧 = 泛指 AI 生成的连续剧集（含真人感/仿真人视频）。部分来源把两者混用，报告里会尽量分开。

---

## 1. 全链路流水线与各环节工具/模型

2026 年业界已收敛到一条高度一致的 6 段流水线，多个独立来源（开源技能集 drama-skills、shuohao-skills、一人创作者调研）描述的是同一骨架：

**想法/改编 → 剧本+剧集圣经 → 冻结角色与场景参考图 → 分镜/镜头表 → 分段生成视频 → 配音+剪辑成片**

### 1.1 剧本（大纲/改编/分集剧本）

| 工具/模型 | 用法 | 来源 |
|---|---|---|
| 豆包（Doubao 2.0） | 中文剧本、大纲、改编，漫剧创作者最常用入口 | [豆包+有戏AI实操](http://www.360doc.com/content/26/0311/03/57798620_1171158043.shtml) |
| DeepSeek / Claude / GPT | Agent 驱动的 Skill 流水线里负责编剧（开源的 drama-skills/shuohao-skills 都跑在 Claude Code / Codex 上） | [drama-skills README](https://github.com/zenstory-ai/drama-skills/blob/HEAD/README_EN.md)、[shuohao-skills](https://github.com/liuqianhonga/money-ideas/blob/HEAD/src/projects/2026-08-31-shuohao-ai-shortdrama-skills.md) |
| 番茄/网文平台 IP 库 | 成熟网文 IP 改编是 2026 下半年爆款主流（红果 S 级头部 AI 漫剧超六成来自网文 IP 改编） | [awnchina 2026下半年盘点](https://awnchina.cn/%e5%91%8a%e5%88%a5%e6%a6%82%e5%bf%b5%e9%a9%97%e8%af%81%ef%bc%9a2026-%e4%b8%8b%e5%8d%8a%e5%b9%b4-aigc-%e5%bd%b1%e8%a6%96%e9%80%b2%e5%85%a5%e5%95%86%e6%a5%ad%e5%8c%96%e5%85%8f%e7%8f%be%e6%96%b0%e9%80%b1/) |

### 1.2 角色与场景设计（"冻结参考图"环节）

这是整条流水线公认的 make-or-break 点。行业共识是：**先批准并冻结角色圣经（脸/服装/声音/禁改项），再去生成几十个镜头**；跳过这步 = 主角脸在集与集之间漂移、约 25% 镜头要重跑。

| 工具/模型 | 用法 | 来源 |
|---|---|---|
| 即梦 AI（Seedream 5.0 / 即梦生图） | 角色卡、角色三视图/表情库、场景氛围图；Seedance 2.0 参考图建议 5-8 张/角色（面部特写、3/4半身、全身、表情、标志性动作），≥1024×1024、纯色背景 | [Seedance 2.0 素材清单](https://github.com/mr-salticidae/knowledge-base/blob/HEAD/03_prompt模板库/Seedance2.0_素材准备清单.md) |
| Midjourney（cref+sref） | 角色一致性：开源 wind-comic 用 cref+sref+8 维"角色 DNA"签名 + 全局 Style Bible 首帧锁风格 | [wind-comic](https://github.com/jsap0914/wind-comic) |
| Nano Banana 2 / Nano Banana Pro | 角色形象重绘/一致性编辑（Higgsfield 聚合平台内提供无限额度档） | [Higgsfield review 2026-09-28](https://justbeingresourceful.com/2026/09/28/higgsfield-ai-review-one-subscription-for-15-video-models-a-production-stack-and-a-valuation-that-says-it-s-not-messing-around/) |

**经验规则**（已验证，多位创作者反复提到）：不要把"白底 T-pose 三视图"喂给视频模型，模型会把白底站姿当成角色特征学进去导致飘；视频参考图要用"生活化场景、3/4 角度、动态姿势"。

### 1.3 分镜

| 工具/模型 | 用法 | 来源 |
|---|---|---|
| Agent Skill（drama-skills 11 个 skills、shuohao-skills） | 自动切分镜：小说→分镜列表→图片提示词→视频提示词，每个环节带脚本化质量检查（角色不雷同、时长精准、风格一致）；"生成前确认闸"：先把本批任务的镜头数/参考/参数列出，人点头后才花额度调用 API | [drama-skills](https://github.com/zenstory-ai/drama-skills/blob/HEAD/README_EN.md) |
| 有戏AI / 魔因漫创（Moyin Creator） | 一站式：剧本粘贴→智能分镜→分镜描述列表可人工修改→批量生成分镜图；魔因漫创支持 Seedance 2.0 多镜头合并叙事（多分镜组合并生成连贯视频，≤9图+≤3视频+≤3音频） | [豆包+有戏AI实操](http://www.360doc.com/content/26/0311/03/57798620_1171158043.shtml)、[魔因漫创介绍](https://github.com/wenjunyun123/personal-knowledge-base/blob/HEAD/笔记同步助手/2026-07-25/GitHubDaily-%20AI%20短剧赛道爆火，自己想动手时，却发现剧本拆分、角色设计、生图、生视频这些环节.md) |
| LTX Studio | 粘贴剧本→自动拆场景/分镜→逐镜头生成编辑；自带 LTX 开源模型 + Veo 2/3.1、Kling 2.6/3.0 高阶档 | [higgsfield.ai 2026 测评](https://higgsfield.ai/blog/best-affordable-ai-video-generators) |

### 1.4 配音（含多角色、口型同步）

| 工具/模型 | 用法 | 来源 |
|---|---|---|
| 火山引擎豆包语音 / 阿里百炼 / ElevenLabs / Azure Speech | 主流 TTS，按角色选音色；单人创作者常用火山或百炼（百炼 VideoRetalk 做对口型） | [smartsub 配音文档](https://github.com/mu-l/smartsub/blob/HEAD/docs/docs/features/tts-dubbing.md)、[yizhi-chengzi/video-ai-talking](https://github.com/yizhi-chengzi/video-ai-talking/blob/HEAD/README.md) |
| 本地免费方案 | Kokoro 多语 v1.1（103 音色）、VITS 中文 AIShell3（174 音色）、ZipVoice 声音克隆、Edge TTS | [smartsub](https://github.com/mu-l/smartsub/blob/HEAD/docs/docs/features/tts-dubbing.md) |
| Seedance 2.0/2.5 原生音画同步 | "双分支扩散变换器架构"，视频+音效+BGM+台词毫秒级同步、口型对齐，可省去大部分后期 | [Seedance 2.0 梳理](http://www.360doc.com/content/26/0214/01/17132703_1170041828.shtml) |

**血泪教训**（2026-09 开源项目实战记录）：音色必须在"角色圣经落库"时就写死（voiceId），而不是配音时现算——否则同一角色在不同镜会拿到不同音色；默认不设音色会导致"十个角色一个甜美女声"，这是观众三秒判定"这是 AI 做的"最刺眼的破绽。旁白音色要排除在角色分配池外。

来源：[ai-comic-drama 音色自动分配 commit](https://github.com/xiangbo1997/ai-comic-drama/commit/f93961f468a7aac244628cd0fe0fae365eb2abc8)

### 1.5 成片（视频生成+剪辑）

**视频生成模型（2026 年 9 月现状）**：

| 模型 | 特点 | 来源 |
|---|---|---|
| Seedance 2.0 / 2.5（字节） | 漫剧/短剧赛道事实标准：4 模态输入（文本/图/视频/音频）、单次 9 图+3 视频+3 音频参考、60 秒 2K 连贯视频、原生音画同步、多镜头自动分镜；掌阅"泡漫"、中文在线等平台级接入 | [Seedance 2.0 梳理](http://www.360doc.com/content/26/0214/01/17132703_1170041828.shtml)、[素材清单](https://github.com/mr-salticidae/knowledge-base/blob/HEAD/03_prompt模板库/Seedance2.0_素材准备清单.md) |
| 可灵 Kling 3.0 / Kling o1（快手） | 写实向强；快手"漫创"依托可灵技术 | [行业深度报告](https://github.com/dzh123553/knowledge-base/blob/HEAD/行业资料/AI漫剧_行业深度研究报告_20260427.md) |
| Sora 2、Veo 3.1、Wan 2.6/2.7、HunyuanVideo 1.5 | 海外/开源阵营；HunyuanVideo 1.5 被多位独立开发者评为"质量最佳"，LTX-Video 为"速度最佳" | [2026 storytelling 工具榜](https://reveriepage.com/blog/top-10-ai-video-generators-for-storytelling-short-films-in-2026-best-tools-for-narrative-creators)、[Chris_Defi 榜单](https://github.com/quriosity-agent/articles/blob/HEAD/2026-03-18/chris-defi-2033832631083966913-analysis-en.md) |
| MiniMax 海螺 | 国产视频模型，wind-comic 等开源管线多引擎竞跑之一 | [wind-comic](https://github.com/jsap0914/wind-comic) |

**聚合/生产平台（一次订阅多模型 + 一致性工具）**：

| 平台 | 特点 | 来源 |
|---|---|---|
| Higgsfield | 15+ 模型聚合（Sora 2 / Veo 3.1 / Kling 3.0 / Seedance 2.5 / WAN 2.6）；Soul ID 跨模型锁角色脸；Cinema Studio 70+ 运镜预设；Genjutsu（2026-09 发布）视频局部重绘；Supercomputer（2026-06）Agentic 任务规划，渲染前先报价格 | [Higgsfield review 2026-09-28](https://justbeingresourceful.com/2026/09/28/higgsfield-ai-review-one-subscription-for-15-video-models-a-production-stack-and-a-valuation-that-says-it-s-not-messing-around/) |
| LTX Studio（Lightricks） | Script-to-storyboard 全流程；自带 LTX 开源模型 + Veo/Kling 高阶档；Standard $35/月起含商用授权 | [higgsfield.ai 2026 测评](https://higgsfield.ai/blog/best-affordable-ai-video-generators) |

**剪辑/字幕**：万兴喵影/Filmora（已集成 Seedance 2.0 插件）、剪映、ffmpeg（字幕烧录：把对白从视频 prompt 里剥离、加 `--no text` 负向提示，后期用 ffmpeg 烧录真实 CJK 字幕——解决 AI 视频里中文乱码的老坑）；开源浏览器剪辑 OpenCut、Remotion（代码化剪辑，可嵌入产品）。

---

## 2. 代表性平台与开源项目清单

**一站式商业/半商业平台**：即梦 AI（字节，Seedance 2.0 入口）、有戏 AI（剧本粘贴→成片）、魔因漫创 Moyin Creator（生产级，剧本→角色→场景→导演→Seedance 2.0，S 级板块）、LTX Studio、Higgsfield、OpenDrama、MagicLight、Topview、PopShort。

**开源 Agent 流水线**（2026 年趋势：从"工具"转向"会调 Agent 干活"的 Skill）：

- [drama-skills](https://github.com/zenstory-ai/drama-skills/blob/HEAD/README_EN.md)（zenstory-ai，2026-09 活跃）：11 个 Agent skills，小说→成片；创意事实全部存 Markdown（剧本.md/视觉设定.md/分镜.md/图片提示词.md/视频提示词.md/剪辑单.md），"改文件=改决策层"；所有花钱的 API 调用先落盘预览、人工确认后才执行。
- [wind-comic](https://github.com/jsap0914/wind-comic)（MIT）：多智能体管线（一句话→成片）：Director→Writer（McKee 结构）→Style Bible 锁风格→角色 DNA→Storyboard（Vision Audit <70 自动重跑）→多引擎视频竞跑（Minimax/Veo/Kling）→Editor 烧录字幕。Provider 无关。
- [shuohao-skills](https://github.com/liuqianhonga/money-ideas/blob/HEAD/src/projects/2026-08-31-shuohao-ai-shortdrama-skills.md)（Apache-2.0，2.4k star）：小说改编大纲→角色设定→美术设定→剧本→分镜切分，脚本化质检。
- BigBanana AI Director（"Script-to-Asset-to-Keyframe" 工业流）、Toonflow（小说→视频全管线）、Komiko（AI 漫画/漫剧）、Jellyfish AI 短剧工厂（全局种子+资产复用）、LocalMiniDrama（全本地部署，隐私向）。

---

## 3. 出圈案例（2026）

| 案例 | 数据 | 说明 |
|---|---|---|
| 《后西游记》 | 30 集，2026-08-31 登陆湖南卫视黄金档+芒果 TV | 国内首部上星全 AI 长剧（无真人、无实景），芒果自研"芒果灵创"平台；首播收视同时段省级卫视第一；广电"边制作、边审核、边播出"新模式标杆 |
| 《砚边青梅》 | 2026-09 出圈；单人创作者"晚晚"20 天完成剧本/建模/画面/配音/剪辑全链路；红果播放破亿、抖音话题阅读 2 亿；带动西安碑林博物馆文旅联动 | "一人公司"创作范本，历史题材 |
| 《波斯复仇记》 | 成本 3000 元，72 小时 GMV 50 万美元（YourChannel） | 海外短剧圈 AI 降本神话级案例（待验证：为行业知识库二手引用，非一手财报） |
| 《我在末世开超市》（灵境万维） | 上线 5 天播放 3 亿、收入 1200 万元，制作成本仅 15 万元 | 武汉科技局 2026-03-24 披露；AI 漫剧 ROI 标杆 |
| 《万妖图录传》 | 连续十一季，全系列累计播放 60 亿 | IP 系列化运营代表 |
| 《废品布衣八零捡宝人》（钱多漫剧） | 第一部收藏 106.4 万，已更新至第十部，红果热度 4500 万+ | 同 IP 对比：龙版传媒《穿越1988》1.2 亿播放但仅产生 7.5 万元收入（2026-09 财报披露），说明**播放≠收入** |
| ReelShort《The Rise of the Lycan Queen》 | 单周约 456 万美元 | 海外 AI 短剧收入天花板参考 |
| 灵矩动漫 | 2025-05 启动，年底月产 50-70 部，预计 2026-03 达 150 部/月 | 工业化产能标杆 |
| 与光同尘 | 3 人团队扩张到 300 人，靠 AI 漫剧实现千万营收 | 工作室增长路径 |

**行业水位**：2026 上半年抖音新上 AI 剧/漫剧 22.19 万部，播放破亿的仅 1055 部（0.47%）；2026 一季度微短剧 12.8 万部中 AI 占比超 95%。DataEye 预测 2026 全年国内 AI 剧+AI 漫剧市场规模突破 400 亿元（+138%）；海外微短剧市场 2026 预计 60 亿美元。**马太效应极强：0.47% 的破亿率，且亿级内部流量分化巨大。**

---

## 4. 成本与周期数据（尽量找真实披露）

### 4.1 漫剧成本/周期演进（中银证券 2026-02 测算 + 36氪/武汉科技局）

| 指标 | 传统动漫 | AI 漫剧（早期） | AI 漫剧（Seedance 2.0 后） |
|---|---|---|---|
| 单集/分钟成本 | 百万级 | 1-1.5 万元/分钟 | 千元/分钟 |
| 单部（90 分钟） | 数千万元 | 150 万元 | 10-50 万元 |
| 制作周期 | 1-2 年 | 1-2 个月 | 10-20 天 |
| 团队规模 | 百人级 | 8-10 人 | 3-5 人 |

来源：[AI 漫剧行业深度研究报告 2026-04-27](https://github.com/dzh123553/knowledge-base/blob/HEAD/行业资料/AI漫剧_行业深度研究报告_20260427.md)

### 4.2 短剧多档成本对比（新浪 2026-09 整理多家媒体报道）

| 制作方式 | 典型成本 | 周期 | 人员 |
|---|---|---|---|
| 真人短剧（传统） | 28-50 万元 | 30-60 天 | 几十人剧组 |
| AI 短剧（流水线型） | 5-7 万元 | 3-7 天 | 1-3 人 |
| AI 短剧（精品型） | 10-20 万元 | 20-30 天 | 3-7 人 |
| AI 短剧（极限低成本） | 3000 元 | 5 天 | 3 人（播放超 5 亿但质量粗糙） |

另：海外知识库口径 AI 短剧 2-3 万元/部、周期 7-10 天；AI 仿真人短剧 1-1.2 万元/部、3 人小组月产 3-4 部；单集成本 5000 元以内、80 集 40 万以内、周期 3-7 天。

来源：[新浪 2026-09](https://k.sina.cn/article_7879922982_1d5ae15260680bbf76.html?from=tech)、[dsh-directorx 海外知识库](https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/74-drama-overseas-loop/drama-overseas-loop.md)、[泡沫前夜 2026-07](https://github.com/yuyangzi/contentcreationkit/blob/HEAD/content/article/20260716-AI短剧泡沫破裂前夜-伪内容淘汰与活人溢价.md)

### 4.3 必须正视的另一面（2026 下半年行业转向）

- **90% 的 AI 漫剧公司处于亏损**（虎嗅 2026-03-09）：流量成本（投流）侵蚀收入，投流占收入 60-80%；尾部作品 ROI 打不正；平台政策变动（红果收紧保底）导致承制公司批量倒闭。
- **平台转向真人**：抖音 2026-04 宣布 5 亿元真人短剧专项扶持，5 月年度真人保底超 15 亿元（部均 +60%）；春节档 5 部破 10 亿爆款全是真人剧。平台数据：流量和付费在涌向真人内容，AI 内容边际收益急速下降。
- **监管收紧**：2026-04-01 AI 漫剧/短剧"先备案后上线"；2026-07-01 AI 微短剧分类分层（≥80 万=重点/30-80 万=普通/<30 万=其他），每集明显位置加 AI 标识，未标注下架；年中累计下架超 7 万部；平台专项整治"千剧一脸"/高频 AI 脸同质化。
- **结论**：2026 下半年的竞争逻辑已从"拼生成速度、拼低成本量产"转向 **IP 价值 + 工业化生产 + 全球化出海**。单纯做"便宜量产"已是红海。

来源：[awnchina](https://awnchina.cn/%e5%91%8a%e5%88%a5%e6%a6%82%e5%bf%b5%e9%a9%97%e8%af%81%ef%bc%9a2026-%e4%b8%8b%e5%8d%8a%e5%b9%b4-aigc-%e5%bd%b1%e8%a6%96%e9%80%b2%e5%85%a5%e5%95%86%e6%a5%ad%e5%8c%96%e5%85%8f%e7%8f%be%e6%96%b0%e9%80%b1/)、[dsh-directorx](https://github.com/laplaceyoung/dsh-directorx/blob/HEAD/knowledge/74-drama-overseas-loop/drama-overseas-loop.md)、[泡沫前夜](https://github.com/yuyangzi/contentcreationkit/blob/HEAD/content/article/20260716-AI短剧泡沫破裂前夜-伪内容淘汰与活人溢价.md)

---

## 5. 已验证 vs 待验证/传闻

**已验证**（多源交叉或一手披露）：
- 6 段标准流水线及"先冻结角色圣经再生成"是行业共识（开源项目 + 创作者实操 + 商业平台教程三方一致）。
- Seedance 2.0（2026-02 发布）的多模态输入/60s 2K/原生音画同步能力（字节官方口径 + 第三方实测清单）。
- 抖音 2026 上半年 22.19 万部 AI 剧/漫剧、破亿率 0.47%（DataEye，经新浪财经引用）。
- 成本演进表（中银证券 2026-02 测算，经行业研报引用）。
- 2026-04 起"先审后播"、7 月 AI 标识+分类分层、平台整治"千剧一脸"（多家媒体 + 广电文件口径）。
- 《后西游记》上星、《砚边青梅》单人 20 天出圈（2026-08/09 行业媒体）。
- 龙版传媒《穿越1988》1.2 亿播放但 6 月仅 80 元、7 月 7.5 万元收入（上市公司公告/董秘办解释，2026-09）。

**待验证/传闻**（单源、二手引用或营销口径，请勿直接作为决策依据）：
- 《波斯复仇记》"3000 元成本、72 小时 50 万美元 GMV"——海外知识库二手引用，无一手财报。
- "一部破 5 亿播放的 AI 短剧成本仅 3000 元、3 人 5 天完成"——媒体转述的"公开报道"，无具名信源。
- Seedance 2.0 营销号口径（"物理误差 3%"、"生成效率提升 30%"、"成功率超 90%"）——多为 360doc/自媒体转述，非官方白皮书。
- LTX Studio/Higgsfield 的具体价格档——来自第三方测评/博客（2026-09），官方可能已调价，选型前请以官网为准。
- 2026 全年 400 亿市场规模——DataEye 预测，非实际结算数据。

---

## 6. 对 video-factory-lab 漫剧线的直接建议

1. **环节划分抄"6 段+确认闸"，不要抄"一键成片"**：剧本→角色圣经冻结→分镜→分段生成→配音→剪辑，每段产物落盘（Markdown/JSON），下一段只读上一段的冻结产物。最关键的是**生成前确认闸**——把本批要花钱的镜头清单（数量/参考图/参数/预估额度）列出来，人点头才执行。这是 drama-skills、wind-comic、行业调研三方一致的省钱方案，也是我们"好莱坞式前置"理念的直接落地。
2. **工具选型：视频生成押注 Seedance 2.x（国内漫剧事实标准），剪辑/字幕走 ffmpeg+代码化管线**：Seedance 2.0 的 9 图参考+首尾帧+原生音画同步正是为漫剧/短剧设计的；但字幕必须后烧（prompt 里剥离对白文本+负向提示），别让模型画中文。配音用火山/百炼 TTS，音色在角色圣经落库时写死 voiceId。剪辑层建议用 Remotion/ffmpeg 代码化而非纯 GUI，保证可复现、可批量。
3. **质量控制点设三处**：① 角色圣经冻结评审（脸/服装/声音/禁改项，人审）；② 分镜 Vision Audit（单镜头视觉打分，低于阈值自动重跑，如 wind-comic 的 <70 分机制）；③ 成片前"AI 味"检查——多角色音色不能同声、中文无乱码、连续性（服装/场景）抽查。平台正在整治"千剧一脸"，角色差异化是过审红线。
4. **成本按"精品档"做预算**：流水线型 5-7 万/部是红海价格战区间；建议按精品型（10-20 万/部、3-7 人、20-30 天）规划，差异化打在剧本和角色设计上。单集 API 成本控制在数千元内是可行的，但别信"3000 元爆款"神话——那是幸存者偏差，破亿率只有 0.47%。
5. **合规前置进流水线**：2026-07-01 起每集必须加 AI 生成标识、先备案后上线；角色形象避免撞脸真人（肖像权）、BGM 用正版授权。建议在"冻结"环节后加一个合规检查清单，作为出厂门禁，否则有下架风险（年中已下架超 7 万部）。

---

*报告完。原始搜索抓取的页面文本仅为公开资料摘要；涉及采购/签约前，请以官网与一手信源复核。*
