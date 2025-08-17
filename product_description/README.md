AI Product Page Generator

This repo lets you generate a polished **HTML product description** for a kids’ football kit **using only two files**:

1. `input.json` – you fill this from your **Google Sheet** variables  
2. `master_prompt.txt` – the AI prompt/template that controls structure and tone  

Optionally, include `trustpilot_reviews_kfk.csv` for real customer quotes.  

No code required. You just provide the JSON + prompt to the AI and it returns the final HTML.

---

## 📂 Repository Structure

├─ input.json # you update this from your Google Sheet
├─ master_prompt.txt # the AI prompt/template
├─ trustpilot_reviews_kfk.csv # optional: reviews for the AI to summarise
└─ README.md # this guide



---

## 🚀 Quick Start (AI-only workflow)

1. **Open your Google Sheet** and copy the latest product variables.  
2. **Paste those values into `input.json`** (see fields below).  
3. **Give the AI** both files:  
   - `master_prompt.txt`  
   - `input.json`  
   *(Optionally also attach `trustpilot_reviews_kfk.csv`.)*  
4. Ask the AI:  

   > Run the prompt in `master_prompt.txt` using `input.json` (and reviews CSV if attached) and output **HTML only**.  

5. Copy the returned **HTML** into your website or CMS.

---

## 📝 Filling `input.json` from your Google Sheet

| Field | Example | Notes |
|---|---|---|
| `seed` | `43` | Selects layout: `(seed % 5) + 1`. Change seed to vary order. |
| `product_name` | `"Inter Miami Away Kids Football Kit 2025/26 (With Socks)"` | Title on page. |
| `team_name` | `"Inter Miami"` | Used throughout. |
| `league_name` | `"MLS"` | Used in Related section. |
| `season` | `"2025/26"` | Mentioned in copy. |
| `kit_type` | `"Away"` | Home/Away/Third. |
| `audience` | `"Kids Kit"` | In **Key Features only**, becomes “Shirt and Shorts”. |
| `socks_included` | `true` | Adds “with socks / with no socks”. |
| `personalisation_available` | `true` | Adds a line about name/number printing. |
| `team_facts` | `true` | Enables the **About Team** section. |
| `player_name` | `"MESSI 10"` | Number ignored → `"Messi"`. Empty string disables section. |
| `max_phrase_overlap` | `0.35` | Encourages fresh wording. |
| `review_file` | `"trustpilot_reviews_kfk.csv"` | If provided, AI selects 3 reviews. |
| `legal_notice` | `"This is a high-quality fan-version shirt/kit."` | Compliance line. |
| `delivery_notes.uk` | `"7–12 working days"` | UK delivery. |
| `delivery_notes.eu` | `"8–14 working days"` | EU delivery (optional). |
| `slogan` | `[ "...", "...", ... ]` | One is chosen under the title. |
| `links.home_url` | `"https://kidsfootballkit.co.uk/"` | Related links. |
| `links.other_leagues_url` | `"https://kidsfootballkit.co.uk/product-category/other-leagues/"` | Optional. |
| `links.league_url` | `"https://kidsfootballkit.co.uk/product-category/other-leagues/inter-miami/"` | Optional. |
| `brand_voice_rules` | `[ "Warm...", "British English...", "Never use 'official'." ]` | Style rules, not printed. |

**JSON rules:**  
- Use plain quotes `"` (no curly quotes).  
- No comments allowed.  
- Empty `player_name` → Player section hidden.  

---

## 🤖 What the AI Handles Automatically

- **Layout**: picked from seed `(seed % 5) + 1`.  
- **CTAs**:  
  - CTA #1 under **Title**.  
  - CTA #2 as the **final line in Description**.  
- **Benefits**: writes 3–5 parent-focused `<li>` items.  
- **Package includes**: `"Kids Kit"` becomes `"Kids Shirt and Shorts (with/without socks)"` in Description.  
- **About the Player**: 3–5 factual, engaging sentences (ignores squad number).  
- **About the Team**: 3–5 light, factual lines if enabled.  
- **Reviews**: 3 customer quotes (≥1 recent, clean text, summarised).  
- **Compliance**:  
  - British English.  
  - Warm, family-friendly tone.  
  - Never uses “official/authentic/licensed/genuine/fake”.  
  - No made-up fabric tech.

---

## 📄 `master_prompt.txt` Structure

1. **Global Rules**  
   - Tone, banned words, compliance, CTA policy.  

2. **Section Definitions**  
   - TITLE (h1 product name, slogan, CTA #1)  
   - BENEFITS  
   - DESCRIPTION (package, legal notice, CTA #2 at end)  
   - DELIVERY  
   - PAYMENT  
   - CONTACT  
   - TEAM (optional)  
   - PLAYER (optional)  
   - REVIEWS  
   - RELATED  

3. **Layout Control**  
   - 5 variants (1..5).  
   - `seed` decides which is used.  
   - TITLE always first.  
   - AI is told: *“If you do not follow layout order exactly, the output will be rejected.”*

---

## ⭐ Reviews CSV

- File: `trustpilot_reviews_kfk.csv`  
- Columns: `reviewer_name`, `review_date`, `review_text`, `rating`, `page`  
- Dates can be “Updated 5 Dec 2024” etc. AI is instructed to clean/summarise.  
- At least 1 review must be recent (≤2 months).  
- Prompt guides AI to avoid artefacts (like `\1`).  

---

## ✅ Do’s & Don’ts

**Do**  
- Fill `input.json` from your Google Sheet.  
- Change `seed` to vary layout.  
- Leave `player_name` empty if no player section needed.  
- Attach CSV for reviews.  

**Don’t**  
- Don’t edit/remove section markers in `master_prompt.txt`.  
- Don’t add banned terms (*official/authentic/licensed*).  
- Don’t include unparseable JSON (curly quotes, comments).  

---

## 🔧 Troubleshooting

- **Placeholders showing up** → Check you gave AI both files.  
- **Wrong layout order** → Ensure `seed` is set; AI picks `(seed % 5) + 1`.  
- **Player missing** → `player_name` empty.  
- **Weird review characters** → Clean CSV or trust AI’s cleaning rules.  

---

## 🗣️ Example AI Command

Please read input.json and master_prompt.txt
(and trustpilot_reviews_kfk.csv if attached),
then run the template to produce the final HTML only.
Follow the layout from the seed, include two CTAs
(Title + Description), keep tone warm with British English,
avoid banned terms, and ensure reviews are clean and readable.



---

That’s it! ✨ Copy variables from your **Google Sheet → input.json**, attach `master_prompt.txt` (+ reviews if needed), and let the AI generate a compliant, ready-to-publish HTML product page.


