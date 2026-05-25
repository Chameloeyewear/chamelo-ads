# Chamelo Ad Pipeline — Claude Code Setup

This repo is the source of truth for Chamelo's AI-driven ad creative pipeline.
When you open this folder in Claude Code, this file loads automatically.

---

## What this pipeline does

Generates static images, motion graphics, and video ads for Meta (Feed, Stories, Reels),
TikTok, and YouTube using AI tools. Output goes to the team review gallery at
**chamelo-gallery.vercel.app** where the team votes and leaves feedback.

---

## One-time setup (do this before anything else)

### 1. Install Claude Code
```
npm install -g @anthropic-ai/claude-code
```

### 2. Install the Higgsfield plugin
In Claude Code, run:
```
/plugins
```
Search for **Higgsfield** and install it. Then authenticate with the shared
Higgsfield team account (ask Chase for credentials).

### 3. Add your kie.ai API key
Create a `.env.local` file in this folder:
```
KIE_AI_KEY=<ask Chase for the team key>
```
This file is gitignored — never commit API keys.

### 4. Install the Vercel CLI + link the gallery
```
npm install -g vercel
cd ../chamelo-gallery
vercel link
```
Use the **chamelo** Vercel team when prompted.

### 5. Clone both repos
```
git clone https://github.com/Chameloeyewear/chamelo-ads.git
git clone https://github.com/Chameloeyewear/chamelo-gallery.git
```

---

## The playbook — read this first

**`03_static-ad-playbook.md`** is the single source of truth for:
- Hard rules (what never appears on an ad)
- Lens technology language (liquid crystal, NOT photochromic, NOT electrochromic)
- Correct product description and pricing
- Per-ad feedback log from previous runs
- All 40 ad template formats
- Tool routing (which AI engine for which ad type)
- Pre-export checklist

Read it before generating anything. Reference it constantly.

---

## Tool routing (quick reference)

| Ad type | Tool | Why |
|---------|------|-----|
| Any ad where glasses are visible | Higgsfield `gpt_image_2` + `medias` product reference | Only way to get correct shield frame |
| Pricing cards, text-only, quote cards | kie.ai `gpt-image-2-text-to-image` | No product needed |
| Image-to-image remixes | kie.ai `gpt-image-2-image-to-image` + `input_urls` | Product accuracy on kie.ai |
| Short video / motion graphics | kie.ai `gemini-omni-video` | 4–10s clips, 9:16 or 16:9 |

---

## Product reference images (Higgsfield media IDs)

Always pass these in the `medias` parameter when generating any ad where the
glasses need to be visible. Without them, AI generates generic wayfarer shapes.

```json
"medias": [{"value": "<ID>", "role": "image"}]
```

| Colorway | Shot | Media ID |
|----------|------|----------|
| Smoke Black | 3/4 hero (primary) | `bcdbeda5-56a7-4c4b-bd10-4f087b7beedf` |
| Smoke Black | Front-on | `7b0b0e55-72e5-468e-81dd-ea8b76d060a5` |
| Black Fire | Hero | `f005fb43-b52d-4823-9f4e-ebc45aba6add` |
| Black Fire | Alt | `4371950e-f289-494e-b258-8ff7c3759f37` |
| Gold | Hero | `63762d19-b95b-4968-b4e2-b7c43a06e021` |
| Gold | Alt | `cc230c18-efac-4222-997f-8a5659765bdc` |
| Knicks | Hero | `5130af25-3e13-4754-b589-7d957e0fd883` |
| Knicks | Alt | `324a966b-633c-497a-b131-2e9e006b6439` |

CDN base for kie.ai `input_urls`:
`https://d2ol7oe51mr4n9.cloudfront.net/user_3D4T4A4nGHmaAJloTbGDOsGdmOh/<ID>.png`

---

## Critical product facts

**The glasses:** Large single-lens sport shield. ONE unbroken shield lens — NO nose bridge,
NO center support bar. Tint SLIDER (not a button, not a dial) running along the RIGHT temple.
Small "Eclipse" logo text on the right temple near the slider. Matte black wraparound frame.

**The lenses:** Liquid crystal / electronically controlled. Changes in 0.1 seconds.
User-controlled — works in any light condition. NOT photochromic (UV-reactive, slow, auto).
NOT electrochromic (ion-based, different tech).

**Approved tint language:** "Instant electronic tint" / "Electronic tint control" /
"Next-gen electronic lenses" / "Liquid crystal lenses"

**Never say:** photochromic / electrochromic / auto-tinting / UV-reactive / Transitions-style

**Pricing (Music Shield Gen 2):** ~~$260~~ → **$219** — Save $41
Always show: strikethrough original + new price + dollar savings. Never show % off.

---

## Generating a batch of ads

1. Tell Claude what campaign you're working on (product + event + ad types needed)
2. Claude reads the playbook and proposes a batch plan
3. Approve, then Claude generates in parallel
4. Review in the gallery at **chamelo-gallery.vercel.app**

---

## Adding ads to the gallery

After generating ads for a new campaign:

```bash
# 1. Compress images (required — raw AI output is too large)
cd /path/to/chamelo-gallery/public/ads
for f in /path/to/new-ads/*.png; do
  sips -s format jpeg -s formatOptions 75 -Z 1200 "$f" --out "${f%.png}.jpg"
done

# 2. Add entries to chamelo-gallery/app/ads.json
# Each entry needs: id, filename, label, product[], event[], type[], medium

# 3. Deploy
cd chamelo-gallery && vercel --prod --yes
```

**ads.json entry format:**
```json
{
  "id": "unique-kebab-case-id",
  "filename": "filename.jpg",
  "label": "Human-readable name",
  "product": ["music-shield-gen2"],
  "event": ["memorial-day"],
  "type": ["feature", "pricing"],
  "medium": "static"
}
```

**Product values:** `music-shield-gen2` | `aura`
**Event values:** `always-on` | `memorial-day` | `fathers-day` | `black-friday`
**Type values:** `feature` | `tech` | `hero` | `social-proof` | `awards` | `pricing` | `dr` | `comparison` | `ugc` | `text`
**Medium values:** `static` | `video` | `motion-graphic`

---

## File structure

```
chamelo-ads/
├── CLAUDE.md                    ← you are here
├── 03_static-ad-playbook.md     ← the rules, always reference this
├── assets/
│   └── products/
│       └── studio-previews/     ← product photography
├── campaigns/
│   └── <campaign-name>/
│       └── AI Selected/         ← approved output goes here
│           └── video/
└── brand-brain/                 ← brand intelligence docs

chamelo-gallery/                 ← separate repo, the review site
├── app/
│   ├── ads.json                 ← add new ad entries here
│   └── page.tsx
└── public/ads/                  ← add compressed images here
```

---

## kie.ai API (direct curl — no MCP needed)

```bash
# Submit a job
curl -X POST "https://api.kie.ai/api/v1/jobs/createTask" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $KIE_AI_KEY" \
  -d '{"model": "gpt-image-2-text-to-image", "input": {"prompt": "...", "aspect_ratio": "4:5", "resolution": "2K"}}'

# Poll for result
curl "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=<ID>" \
  -H "Authorization: Bearer $KIE_AI_KEY"
# Check: data.state == "success", URL in data.resultJson.resultUrls[0]
```

---

## Gallery link

**[chamelo-gallery.vercel.app](https://chamelo-gallery.vercel.app)**

Filter by product / event / type / medium. Click any ad to react and leave notes.
The whole team shares the same feedback state.
