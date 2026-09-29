# Scene Card Template

> One card per scene/location. Frozen before production.

```yaml
scene_id: SCENE_001
name: ""
description: ""     # what the place is, in one paragraph

# --- Visual identity (frozen) ---
environment: ""     # interior/exterior, architecture, nature
lighting: ""        # time of day, key light, mood
art_style: ""       # must match the project's global art style
color_palette: []   # 3-5 hex colors
key_props: []       # objects that must appear when this scene is used

# --- Generation anchors ---
seed: 0
reference_images:
  - assets/scenes/SCENE_001_wide.png
negative_prompt: "changing architecture, inconsistent lighting"

# --- Continuity locks (LOCK-*) ---
# One sentence per lock, BILINGUAL. Copy VERBATIM into every generation prompt.
continuity_locks:
  - id: LOCK-LIGHTING
    zh: "黄昏时分，暖金色侧光，街道有长长的影子"
    en: "dusk, warm golden side light, long street shadows"
  - id: LOCK-ARCH
    zh: "青砖马头墙徽派建筑，门前一对石狮子"
    en: "Huizhou-style gray-brick horse-head walls, a pair of stone lions by the gate"

# --- Versioning ---
version: v1.0
frozen_by: ""
frozen_at: ""
```
