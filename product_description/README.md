# KidsFootballKit Product Page Generator — README

> Generate consistent, human‑sounding **HTML product descriptions** for football shirts/kits by feeding a small JSON payload and a single “master prompt” to your LLM. This repository is designed for deterministic, high‑quality output at scale across teams, seasons, and kit variants.

---

## Overview

This project turns **structured data** (from Google Sheets → JSON) into a **fully‑formed HTML** product description using a carefully engineered prompt (`master_prompt.txt`).

- **Source of truth:** a Google Sheet with team & player info.
- **Per‑product input:** one small `input.json`.
- **Output:** clean **HTML only** (no Markdown, no explanations).
- **Tone:** warm, family‑friendly, British English, and *not robotic*.
- **Safety:** no licensing claims, no fabric tech invention, no “official”.

---

## How it works (high level)

1. **Prepare data in Google Sheets.** Keep one row per product (team, kit type, season, etc.).
2. **Export data to JSON** per product (manually or programmatically).
3. **Run an LLM** (e.g., ChatGPT API, n8n, your agent) with:
   - `master_prompt.txt` as the system/instruction prompt, and
   - your `input.json` injected into `{{json}}` in the prompt.
4. The LLM renders **HTML only**, following the layout and constraints in the master prompt:
   - Sections like **TITLE → BENEFITS → DESCRIPTION → DELIVERY…**
   - Conditional sections for **Team**, **Player**, and **Reviews**.
   - Two CTAs enforced (Title + final line of Description).

The prompt uses a few deterministic rules so outputs vary naturally but stay within a strict structure. Example: `layout_variant = (seed % 5) + 1` selects one of five section orders.

---

## Quick start

1. **Copy** `master_prompt.txt`.
2. **Create** a minimal `input.json` (see template below).
3. **Call your LLM** with:
   - System: contents of `master_prompt.txt`
   - User: set `{{json}}` to the JSON string
4. **Save** the LLM response as `product.html` and publish.

> The model must output **HTML only** (enforced by the prompt).

---

## Preparing your data (Google Sheets → JSON)

Typical columns you’ll want in your sheet (one row per product):
- `product_name`, `team_name`, `league_name`, `season`, `kit_type`, `audience`
- `socks_included` (`true|false`), `personalisation_available` (`true|false`)
- `team_facts` (`true|false`): whether to include the Team section
- `player_name` (optional, e.g., `"MESSI 10"` → number will be dropped in output)
- `delivery_notes.uk`, `delivery_notes.eu`
- `links.home_url`, `links.league_url`, `links.other_leagues_url`
- `review_file` (CSV path)
- `seed` (integer, e.g., 1, 2, 3…) and `max_phrase_overlap` (0–1 float)
- `legal_notice` (short disclaimer line)
- `slogan` (array of one‑line trust messages)
- `brand_voice_rules` (array of short rules)

Export each row to a **separate JSON** object file and pass it to the LLM as `{{json}}`.

---

## `input.json` — fields & what they do

Below is a **complete reference** of configurable variables found in `input.json`.

### Core identity
- **`product_name`** *(string, required)* — Appears in headings; avoid excessive repetition elsewhere.
- **`team_name`** *(string, required)* — Used across sections; drives “Team” content if `team_facts` is true.
- **`league_name`** *(string, required)* — Used in related links copy (label).
- **`season`** *(string, optional)* — If absent, the prompt avoids stating a season.
- **`kit_type`** *(string, required)* — e.g., `"Home"`, `"Away"`, `"Third"`.
- **`audience`** *(string, required)* — e.g., `"Kids Kit"`, `"Men"`, `"Women"`.

### Options & flags
- **`socks_included`** *(bool)* — Controls the “Package includes” bullet in **Description** (`"Shirt and Shorts"` wording here only; appends “with socks / with no socks”).
When generating the "What’s Included" list, always follow these rules:
        If product_name has “Kit” → list Shirt + Shorts.
        If it also has “(With Socks)” → include Socks.
        If it has “(No Socks)” → exclude Socks.
        If product_name has “Shirt” (not Kit) → Shirt only.
Do not invent extra items or wording.
- **`personalisation_available`** *(bool)* — Enables a personalisation bullet in **Description** (with safe wording—no promises beyond availability).
- **`team_facts`** *(bool)* — If true, shows **About the Team** with factual but non‑invented content.
- **`player_name`** *(string, optional)* — If provided, shows **About the Player**; the prompt will drop digits (e.g., `"MESSI 10"` → “Messi”) and use story‑like phrasing.

### Reviews
- **`review_file`** *(string, required for Reviews)* — CSV path used to **select 3 reviews**, at least one from the **last 2 months**. The prompt expects columns for *reviewer name*, *date*, and *text*. Ensure your runtime can access the file and your column names map cleanly.

### Compliance & voice
- **`legal_notice`** *(string)* — Short, human‑sounding notice (e.g., *“high‑quality fan‑version shirt/kit.”*).
- **`brand_voice_rules`** *(array of strings)* — Reinforces tone (warm, British English, no “official”, etc.).
- **`max_phrase_overlap`** *(number)* — Soft cap for repeated phrasing across products (the LLM treats this as guidance).

### Delivery & links
- **`delivery_notes.uk`** *(string)* — Required (e.g., `7–12 working days`).  
- **`delivery_notes.eu`** *(string, optional)* — If present, adds an EU line.
- **`links.home_url`**, **`links.league_url`**, **`links.other_leagues_url`** *(strings)* — Used in the **Related Products** section. Ensure **labels match URLs** (e.g., don’t label “MLS” if the link points to a specific team collection).

### Slogans
- **`slogan`** *(array of strings)* — A pool of trust lines the model can use. Keep them typo‑free (e.g., “transactions”, not “trasactions”).

### Randomness & diversity
- **`seed`** *(integer)* — Drives `layout_variant = (seed % 5) + 1`. You can simply use the **row sequence number** starting from 1.
- **`layout_variant`** *(optional, number 1–5)* — If you provide this explicitly in the JSON, it **overrides** the seed rule.
- **`cta_style`** *(optional, enum)* — If set, forces CTA style to one of: `now_simple`, `gift_deadline`, `low_stock`.
  - If not set, the prompt picks randomly.

---

## Variables & controls **inside** `master_prompt.txt` (not always in JSON)

These can be treated as **prompt‑side knobs**:

- **`layout_variant`** *(1–5)* — Section order. Defaults to `(seed % 5) + 1` when not provided.
- **`cta_style`** — CTA phrasing style. If absent in JSON, the prompt picks one at random.
- **Uniqueness pools** — The prompt references “opener phrasings”, “CTA endings”, and “paragraph cadences”. They are internal variation rules (seed‑driven selection is claimed for consistency; the exact pools are inside the prompt).
- **Humanisation switches** — Sentence length variety; a mix of short, medium, long; casual connectors; an anecdotal line in Benefits; small, natural imperfections.

> You do **not** need to set these in JSON unless you want to force a specific layout or CTA style.

---

## Master prompt structure (section by section)

Each **SECTION** is defined once, and a final **LAYOUT CONTROL** block dictates the output order. Conditionally rendered sections are wrapped with `{{#if ...}}` guards.

1. **TITLE**  
   - H1 with product name; short, warm intro; **CTA #1** (`{{CTA_LINE}}`) under the title.

2. **BENEFITS**  
   - Emotional, parent‑oriented bullets. Mix lengths. Includes a casual anecdotal line.

3. **PRODUCT DETAILS**
Each description must include a Product Details block with the heading:
“{{json.team_name}} Kit: Key Details & Features”
The block should cover:
 Material: Always mention polyester (100% lightweight). Add light benefit wording (comfort, durability, washability). Avoid technical fabric jargon.
 Club Inspiration: Link to shirts worn by {{json.team_name}} players.
 Detail: Mention neckline or small authentic touch.
 Print & Logos: Note that crest/sponsor logos are printed with durable finish.
 Washing: Simple care line, e.g. machine wash cold, air dry.  
     

4. **DESCRIPTION**  
   - Key bullets including *Package includes* (special wording), *Personalisation* (if available), durability/fit, a legal line, and **CTA #2** as the **final** `<li>` in this section.

5. **DELIVERY**  
   - UK and EU timings; a note about potential delays (`{{POSSIBLE_DELAY}}`) and a friendly pointer to delivery details (`{{DELIVERY_DETAILS}}`).

6. **PAYMENT**  
   - Payment methods and a short human line for payment issues (`{{PAYMENT_ISSUE}}`).

7. **CONTACT (“How to Reach Us”)**  
   - Clear support email line with friendly tone. (Make sure your email matches your shop.)

8. **TEAM** *(conditional on `team_facts`)*  
 - Display 4–6 bullets introducing the club in a light, fan-friendly way, 80-100 words
 Do not invent facts. Use only text provided in the club library (team_knowledge.paragraph).
 If no club text is provided, output one neutral support line (no specifics).
 Mix short and long sentences; avoid encyclopedia tone.

9. **PLAYER** *(conditional on `player_name`)*  
 - Output a story-style paragraph about the player, 80-100 words
 Drop the shirt number from the name (e.g., “MESSI 10” → “Messi”).
 Use only text provided in the player library (player_knowledge.paragraph).
 If no player text is provided, output one neutral admiration line.
 Allow a gentle playful touch; keep it human; never invent trophies or stats.

10. **REVIEWS**  
   - Pull **3** short, human‑sounding summaries from `review_file`, including **≥1 recent** (last 2 months). Output reviewer name + date + paraphrased quote.

11. **RELATED**  
    - Links to **home**, **league/team collection**, and **other leagues** with short, helpful lead‑ins.

12. **LAYOUT CONTROL (OUTPUT ORDER)**  
    - Five predefined layout variants. The prompt tells the model to **render sections above** and then **output them in this exact order** for the chosen variant. **TITLE is always first.**

13. **DIVERSITY & UNIQUENESS**  
    - Encourages varied sentence lengths, connectors, and fresh phrasing. Soft cap via `max_phrase_overlap`.

14. **RISK & COMPLIANCE**  
    - No “official”, no licensing/auth claims, no invented fabric tech, and safety around specific measurements.

15. **CTA RULES**  
    - **Two CTAs total**: one under **TITLE**, one as the **final line** of **DESCRIPTION**. Styles: `now_simple`, `gift_deadline`, `low_stock`.

---

## Running at scale

- **Programmatic export from Sheets:** Use Apps Script/Python to export each row as JSON.
- **Automation runner:** n8n, Zapier, or your own script can iterate rows → JSON → LLM call → save `product.html`.
- **Determinism:** Set `seed` to the **row index** for predictable layout variation.  
- **Templating:** Treat `master_prompt.txt` as a single static prompt. Inject the JSON string into `{{json}}` before sending to the LLM.

---

## Best practices & guardrails

- Keep **facts minimal but correct**; if you don’t have specifics, let the prompt write around them.
- Ensure **review CSV** is reachable by your runtime and columns are mappable to *name / date / text*.
- Align **labels with links** in the Related section (don’t label “MLS” if the URL is a team category).
- Maintain **British English** and avoid banned terms (*“official”*).  
- Avoid scarcity claims if your brand guidelines forbid them. If you allow them, `cta_style: "low_stock"` is available.

---

## Example `input.json`

```json
{
  "seed": 1,
  "product_name": "Inter Miami Away Kids Football Kit 2025/26 (With Socks)",
  "team_name": "Inter Miami",
  "league_name": "MLS",
  "season": "2025/26",
  "kit_type": "Away",
  "audience": "Kids Kit",
  "socks_included": true,
  "personalisation_available": true,
  "team_facts": true,
  "player_name": "MESSI 10",
  "max_phrase_overlap": 0.35,
  "review_file": "trustpilot_reviews.csv",
  "legal_notice": "This is a high-quality fan-version shirt/kit.",
  "delivery_notes": {
    "uk": "7–12 working days",
    "eu": "8–14 working days"
  },
  "links": {
    "home_url": "https://kidsfootballkit.co.uk/",
    "other_leagues_url": "https://kidsfootballkit.co.uk/product-category/other-leagues/",
    "league_url": "https://kidsfootballkit.co.uk/product-category/other-leagues/inter-miami/"
  },
  "slogan": [
    "Shop with confidence — secure payments and a satisfaction guarantee on every order."
  ],
  "brand_voice_rules": [
    "Warm, plain-spoken, family-friendly.",
    "British English spelling.",
    "Never use the word 'official'."
  ]
}
```

---

## Troubleshooting

- **Model returns Markdown or explanations.**  
  Ensure the *system prompt* is `master_prompt.txt` and that it clearly states “Output HTML ONLY — no explanations.”

- **Wrong section order.**  
  Check `seed` or explicit `layout_variant`. The model must follow the order block under **LAYOUT CONTROL**.

- **No reviews appear.**  
  Confirm the runtime can access `review_file` and that your CSV has the expected columns and recent rows.

- **Scarcity language conflicts.**  
  If your brand forbids scarcity messaging, remove/override `cta_style: "low_stock"` or set `cta_style` explicitly per product.

- **Link label mismatch.**  
  Ensure `league_name` and `league_url` logically match; otherwise customise the label text in your runner.

---

## FAQ

**Q: Can I force a specific section order?**  
Yes — set `"layout_variant": 1..5` in your JSON to override the seed rule.

**Q: Can I force CTA style?**  
Yes — set `"cta_style": "now_simple" | "gift_deadline" | "low_stock"`.

**Q: Do I need `season`?**  
No. If it’s missing, the prompt avoids stating a season.

**Q: What happens if `player_name` is empty?**  
The **Player** section is skipped.

**Q: Where do I put my support email?**  
Edit the **Contact** section template line in `master_prompt.txt` to match your shop, or inject it as a JSON field if you plan to vary by site.

---

## License

MIT (or your preferred license).

---

## Changelog

- **v1.0.0** — First public release with 5 layout variants, dual‑CTA logic, CSV‑driven reviews, and seed‑based variation.
