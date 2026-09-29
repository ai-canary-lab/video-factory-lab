# 知识库总索引 INDEX

> 维护规则（防腐烂）：
> 1. 新增任何文档，先在这里登记一行；
> 2. 研究报告命名 `NN-主题.md`，按编号顺序；
> 3. 过时文档不删除，标注 `[已过时: YYYY-MM-DD]` 并链接替代文档；
> 4. 案例拆解进 `docs/cases/`，原始调研进 `docs/research/`。

## SOP（agent 行动手册）

| 文件 | 内容 |
|---|---|
| `sop/00-overview.md` | 总览：阶段地图、人机分工总表 |
| `sop/01-preproduction.md` | 前期：创意 → 剧本 → 角色 → 场景 → 分镜 |
| `sop/02-production.md` | 制作：生图 → 视频 → 质检（含高效协作模式） |
| `sop/03-post.md` | 后期：配音 → 拼接 → 字幕 → 交付验收 |
| `sop/gates.md` | 各阶段准入 / 准出门禁清单 |

## 研究报告 `docs/research/`

| 文件 | 内容 | 状态 |
|---|---|---|
| `01-ai-video-landscape-2026.md` | 2026 AI 视频市场格局（商用 API vs 开源） | ✅ 2026-09-29 |
| `02-consistency-techniques.md` | 角色/场景/风格一致性技术 | ✅ 2026-09-29 |
| `03-comic-drama-pipeline.md` | AI 漫剧生产流水线 | ✅ 2026-09-29 |
| `04-marketing-video-pipeline.md` | AI 产品营销视频流水线 | ✅ 2026-09-29 |
| `05-sekoai-overman.md` | SekoAI 与《无敌超人》长片拆解 | ✅ 2026-09-29 |
| `06-hypit.md` | hypit 一键复制爆款视频 | ✅ 2026-09-29 |

## 案例拆解 `docs/cases/`

- `overman.md` — 《无敌超人》Overman：B站 28 分钟级 AI 长片标杆拆解（工具链/资产先行/转场 QC）

## Prompt 模板 `prompts/templates/`

- `character-card.template.md` — 角色卡（冻结用）
- `scene-card.template.md` — 场景卡（冻结用）
- `shot-prompt.template.md` — 分镜 prompt（组装用）

## 脚本 `scripts/`

- 规划中，见 `scripts/README.md`（批量生成、质检、拼接、字幕）
