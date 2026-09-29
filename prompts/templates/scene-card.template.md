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

# --- Versioning ---
version: v1.0
frozen_by: ""
frozen_at: ""
```
