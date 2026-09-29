# Character Card Template

> One card per character. Frozen before production. Nothing here changes mid-project.

```yaml
character_id: CHAR_001
name: ""
role: protagonist | supporting | extra

# --- Visual identity (frozen) ---
appearance:
  age: ""
  gender: ""
  face: ""          # face shape, eyes, nose, mouth, distinguishing marks
  hair: ""          # color, style, length
  body: ""          # build, height impression
outfit:
  default: ""       # the ONE outfit used across the whole project unless script says otherwise
  variants: []      # only if script requires costume changes
art_style: ""       # e.g. "anime cel-shaded, clean lineart" / "photorealistic"
color_palette: []   # 3-5 hex colors that define this character

# --- Generation anchors ---
seed: 0                       # fixed seed for this character
reference_images:
  - assets/characters/CHAR_001_front.png
  - assets/characters/CHAR_001_side.png
negative_prompt: "changing clothes, changing hairstyle, extra limbs, deformed face"

# --- Continuity locks (LOCK-*) ---
# One sentence per lock, BILINGUAL. Copy VERBATIM into every generation prompt.
# gate_check.py verifies each shot prompt contains all locks of its characters.
continuity_locks:
  - id: LOCK-FACE
    zh: "圆脸，左眉上有一颗小痣"
    en: "round face, small mole above the left eyebrow"
  - id: LOCK-OUTFIT
    zh: "藏青色立领长袍，腰间束白色布带"
    en: "dark-blue mandarin-collar robe, white cloth belt at the waist"

# --- Versioning ---
version: v1.0
frozen_by: ""
frozen_at: ""
```

## Usage

1. Fill the card, generate turnaround views (front/side/back), pick the best.
2. Freeze: set `version`, `frozen_by`, `frozen_at`. From here the card is read-only.
3. Every shot prompt references `character_id` + attaches reference images.
