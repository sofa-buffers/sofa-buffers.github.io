# Maintenance Playbook — SofaBuffers Landing Page

> **For Claude:** When the user says **"update the page"** (or similar), read this file and follow it end-to-end. This is the single source of truth for keeping `index.html` in sync with the actual project.

This is a **static, dependency-free** site: plain HTML, CSS and JS in a single `index.html`. There is **no build step and no framework** — never introduce one. Edit `index.html` directly.

---

## 1. Where the truth lives (always re-fetch before editing)

The page describes the SofaBuffers project. The authoritative facts come from these repos — **re-read them every time** so the page reflects reality, not memory:

| Source | URL | What to pull from it |
|--------|-----|----------------------|
| Org overview | https://github.com/sofa-buffers | The list of repositories (languages!), org description |
| Documentation README | https://raw.githubusercontent.com/sofa-buffers/documentation/main/README.md | Feature list, why-it-exists, format comparison |
| Architecture / spec | https://raw.githubusercontent.com/sofa-buffers/documentation/main/ARCHITECTURE.md | Wire types, varint/zig-zag, sequences, API constants, generated-object API |

Use `WebFetch` (or `gh api` when authenticated) to read them. The org member list is private; repo metadata is public.

---

## 2. The "update the page" procedure

1. **Re-fetch** all three sources above.
2. **Diff against the page.** Walk the data points in §3 and check each one still matches the sources.
3. **Edit `index.html`** for anything that drifted. Keep edits surgical.
4. **Keep the mirrored values in sync** — several facts appear in 3–4 places at once (see §3). Update *all* copies or the page contradicts itself.
5. **Validate** (see §5).
6. **Update `README.md`** if the project summary/links changed there too.
7. **Ship it** (see §6) — commit on a branch + open a PR. **Do not merge/deploy to production without explicit user confirmation** (merging to `main` republishes the live GitHub Pages site).

---

## 3. Data points that drift — and every place they appear

When one of these changes, grep for **all** occurrences. The list of languages is the thing most likely to change.

### A. Supported languages / core libraries
The project currently ships **7 language core libraries + a generator**. This count and list appear in **five** places in `index.html`:

1. **Hero stat** — `<span><b>7</b> language core libraries</span>`
2. **Section heading** — `<h2 class="sec-title">One format, seven languages</h2>` (update the spelled-out number)
3. **`.lang-grid`** — one `<a class="lang">` card per corelib + the generator card
4. **Footer columns** — `Core libraries` and `More languages` link lists
5. **JSON-LD** — `"programmingLanguage": [ ... ]` in the `<script type="application/ld+json">` block

…plus the **meta keywords** (`<meta name="keywords">`) and **meta description** mention languages generically — only touch if the framing changes.

> ⚠️ If you change the number of languages, update **all five** spots **and** the spelled-out word in the heading ("seven"). A mismatch is the most common bug here.

### B. Repository links
Every `https://github.com/sofa-buffers/<repo>` link must point to a real repo. They appear in the nav, hero CTAs, `.lang-grid`, the CTA band, and the footer. If a repo is renamed/added/removed, fix every link and the `README.md` link list.

### C. Format / spec facts
From `ARCHITECTURE.md`. These live in the **"Under the hood"** section:
- The **8 wire types** table (`wire-format.txt` code block: `0x0`–`0x7`)
- **Varint** example (`300 -> 0xAC 0x02`) and **zig-zag** examples (`-1 -> 1`, `-2 -> 3`)
- **Field header** formula `(id << 3) | type`
- **API constants** if surfaced (API_VERSION, field-ID range, value domain)
- The **generated-object** code sample (`Person().serialize()` / `deserialize()`)

If the spec revises any of these, update the code blocks to match exactly.

### D. Feature descriptions
The 6 feature cards under **"What makes it cool"** summarize the docs (streaming-first, tightly packed, self-describing TLV, nested sequences, schema→code, runs everywhere). Re-word only if the docs add/drop a headline capability.

### E. SEO / housekeeping
- `sitemap.xml` — bump the implied freshness if you do a big content change (optional; there's intentionally no hard-coded `lastmod`).
- `robots.txt` and `.nojekyll` — usually no change.
- If the project's one-line pitch changes, update `<title>`, `<meta name="description">`, and the Open Graph / Twitter description **together**.

---

## 4. Things that must NOT change (unless the user explicitly asks)

- **The two slogans, verbatim:**
  - `"Because JSON is too fat, and Protobuf is too Google."`
  - `Built for embedded developers and cloud engineers 🚀`
- **No toolchain.** Plain HTML/CSS/JS only — no npm, bundlers, frameworks, or external CDN dependencies.
- **No Claude/AI attribution** anywhere in the committed files or commit messages (the user removed these deliberately).
- **The calm palette.** CSS custom properties at the top of `index.html` (`--accent` teal, `--accent-2` indigo, slate backgrounds) with the light/dark toggle. Don't reintroduce neon/glowing styles.
- **Minimal JS** (theme toggle, copy buttons, scroll-reveal). Keep it dependency-free and respect `prefers-reduced-motion`.

---

## 5. Validation before shipping

Run these checks (no browser needed for the first three):

```bash
# 1. HTML tags balanced
python3 - <<'PY'
import re
h=open('index.html').read()
for t in ['section','article','footer','header','main','div','pre','script','style']:
    o=len(re.findall(r'<%s[\s>]'%t,h)); c=len(re.findall(r'</%s>'%t,h))
    print(t, o, c, '<-- MISMATCH' if o!=c else '')
PY

# 2. No AI/Claude hints leaked into the page
grep -niE "claude|anthropic|generated with|🤖" index.html || echo "clean"

# 3. Every corelib link resolves (needs network / token)
grep -oE "https://github.com/sofa-buffers/[a-z0-9-]+" index.html | sort -u

# 4. Optional visual check — serve and look
python3 -m http.server 8000   # then open http://localhost:8000
```

Confirm the language count is consistent across all five locations from §3A.

---

## 6. Shipping changes

GitHub Pages publishes from the **`main` branch, root path**. So:

1. Branch off `main`, commit with a plain message (**no Co-Authored-By / AI trailer**).
2. Push and open a PR (`gh` or the API). The user supplies a token when needed — **never commit or hard-code a token**, and remind them to revoke it afterward.
3. **Merging the PR to `main` deploys to production** (`https://sofa-buffers.github.io`). Only merge when the user explicitly says so. After merge, Pages takes ~1–4 minutes to rebuild; verify the live site reflects the change (and suggest a hard refresh).

---

## 7. Quick recipe — "a new language was added"

Say the org adds `corelib-kt` (Kotlin). Do all of this:

1. **Hero stat:** `7` → `8`.
2. **Heading:** "seven languages" → "eight languages".
3. **`.lang-grid`:** add a card (copy an existing `<a class="lang">`, pick a distinct `.badge` background color, set the repo URL, name, and `<small>` repo slug).
4. **Footer:** add the link under `Core libraries` or `More languages`.
5. **JSON-LD:** add `"Kotlin"` to the `programmingLanguage` array.
6. **(Optional) meta keywords:** add the language if it helps SEO.
7. Validate (§5) → branch + PR (§6) → ask before merging.

Removing or renaming a language is the same checklist in reverse — touch all the same spots.
