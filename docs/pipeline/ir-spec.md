# IR 设计：双层 JSON 中间表示（v0.1 草案）

> 吸收自：ArcReel（`project.json` 单一真相源）、MoneyPrinterTurbo
> （`VideoParams` + 词级对齐表）、hypit（SVML 词锚定思想）。
> License 注意：ArcReel 为 AGPL-3.0，只吸收思想，不复用代码。

## 层 1：assets.json —— 资产层（冻结）

剧本和分镜**只引用资产 ID，不内联定义**。完整性校验：引用的 ID 必须存在，
否则拒绝入队（fail loud）。

```json
{
  "characters": {
    "CHAR_001": {
      "card": "characters/CHAR_001.md",
      "locks": ["LOCK-FACE", "LOCK-OUTFIT"],
      "voice": "elevenlabs:voice_id_xxx"
    }
  },
  "scenes": {
    "SCENE_001": { "card": "scenes/SCENE_001.md" }
  }
}
```

## 层 2：timeline.json —— 时间轴层（词锚定）

事件**锚定在词上而非秒上**（借鉴 SVML）：改词自动重排时间轴，
不用手工重对。词级对齐表统一为 `word_timings.json`（WhisperX 产出）。

```json
{
  "shots": [
    {
      "shot_id": "EP01_SC03_SH02",
      "characters": ["CHAR_001"],
      "scene": "SCENE_001",
      "camera": { "shot_size": "medium", "movement": "dolly in" },
      "lines": [
        { "text": "推开门。",
          "words": [
            { "w": "推开", "t0": 0.0, "t1": 0.6 },
            { "w": "门。", "t0": 0.6, "t1": 1.0 }
          ] }
      ],
      "takes": 3
    }
  ]
}
```

TTS 按 `{provider}:{voice}` 前缀路由（如 `elevenlabs:xxx`、`fish-audio:yyy`），
provider 先做 TTS / LLM 两类即可，不要一上来就抽象所有模型。

## runs/ 三账本（代替重型队列）

每次运行在 `runs/<timestamp>/` 下写三个 JSON，状态机三态独立：

| 文件 | 记什么 |
|---|---|
| `task_state.json` | 任务状态：queued / running / done / failed |
| `provider_checkpoint.json` | 供应商侧提交状态：submitted = 很可能已计费 |
| `artifacts.json` | 产物有效性：current / stale / missing / blocked；stale 不自动重生 |

断点续跑：`resume: true` 从 checkpoint 继续，不重跑已提交部分。

## 门控

- **阶段门控** `stop_at`：任何阶段都可停下检查再继续（抄 MoneyPrinterTurbo）。
- **花钱门**：调用付费接口前先报价 + 显式确认，确认标志永不静默。
- **批量清单**：一次跑一批，单任务失败不阻塞整批，最后出结构化汇总报告（含 `failed_stage`）。
