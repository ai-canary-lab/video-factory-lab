# Shot Prompt Template

> One entry per shot. Assembled from frozen cards + shot-specific action.
> The model only fills in the ACTION — everything else is copied from cards.

```yaml
shot_id: EP01_SC03_SH02
duration_sec: 10          # fixed ~10s per clip in v0.1
characters: [CHAR_001]    # references to frozen character cards
scene: SCENE_001          # reference to frozen scene card

camera:
  shot_size: ""           # extreme close-up | close-up | medium | wide | aerial
  movement: ""            # static | pan left | dolly in | orbit | handheld
  angle: ""               # eye-level | low | high | dutch

action: ""                # THE ONLY free-text field. One clear action per shot.
mood: ""
audio_mood: ""            # e.g. "quiet piano", "upbeat electronic" (synthesized, no uploads)

# --- Assembly (do not hand-write; generated from cards) ---
assembled_prompt: |
  [art_style from project]
  [character appearance block from CHAR_001 card]
  [scene description block from SCENE_001 card]
  Action: <action>
  Camera: <camera.*>

negative_prompt: "morphing face, changing clothes, extra limbs, watermark, text"
seed: 0
takes: 3                  # generate N takes, keep the best
```

## Naming

Output files: `{shot_id}_take{n}.mp4` → e.g. `EP01_SC03_SH02_take2.mp4`
