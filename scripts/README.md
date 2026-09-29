# scripts/

流水线脚本（待建设）。规划中的脚本：

| 脚本 | 作用 |
|---|---|
| `gen_batch.py` | 按分镜表批量生成：读 `shots.yaml` → 调生图/生视频 → 落盘候选 |
| `qc_frames.py` | 质检：抽帧 + 一致性检查（人脸/服装/场景），输出通过率报告 |
| `concat.py` | 按 EDL 拼接 clips（ffmpeg），统一分辨率/帧率 |
| `subtitle.py` | 配音稿 → TTS → 字幕轴对齐 |
| `shots_schema.json` | 分镜表 JSON Schema（`shot-prompt.template.md` 的机器版） |

约定：
- 所有脚本幂等、可断点续跑。
- 每次运行写 `runs/<timestamp>/` 日志，记录模型/prompt/seed（复现性）。
