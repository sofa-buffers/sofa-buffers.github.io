# Maintenance Playbook — SofaBuffers Landing Page

> **For Claude:** When the user says **"update the page"** (or similar), read this file and follow it end-to-end. This is the single source of truth for keeping `index.html` in sync with the actual project.

This is a **static, dependency-free** site: plain HTML, CSS and JS in a single `index.html`. There is **no build step and no framework** — never introduce one. Edit `index.html` directly.

---

## 1. Where the truth lives (always re-fetch before editing)

The page describes the SofaBuffers project. The authoritative facts come from these repos — **re-read them every time** so the page reflects reality, not memory:

| Source | URL | What to pull from it |
|--------|-----|----------------------|
| Org repo list | `https://api.github.com/orgs/sofa-buffers/repos?per_page=100` | **The canonical core-library list.** Every repo whose name starts with `corelib-` is one card in the language grid — nothing else is. Also gives each repo's one-line `description`. |
| Per-corelib README | `https://raw.githubusercontent.com/sofa-buffers/<corelib-repo>/main/README.md` | The **distinguishing blurb** for each card (the `<small>` line). Read the intro paragraph after the `## SofaBuffers <Lang> library` heading — it says what makes this build different (target, std vs no_std, speed vs size, runtimes). |
| Documentation README | https://raw.githubusercontent.com/sofa-buffers/documentation/main/README.md | Feature list, why-it-exists, format comparison |
| Architecture / spec | https://raw.githubusercontent.com/sofa-buffers/documentation/main/ARCHITECTURE.md | Wire types, varint/zig-zag, sequences, API constants, generated-object API |

Use `WebFetch` / `curl` (or `gh api` when authenticated) to read them. The org member list is private; repo metadata is public.

> **The grid is driven by `corelib-*` repos, not by language.** Some languages ship **more than one** core library targeting different use cases — e.g. `corelib-c-cpp` (embedded C/C++) vs `corelib-cpp` (pure C++20, speed-tuned), and `corelib-rs` (high-speed `std`) vs `corelib-rs-no-std` (`#![no_std]`, microcontrollers). Each `corelib-*` repo gets **its own card**, and its `<small>` blurb is what tells the two builds of the same language apart. The `generator` repo is **not** a corelib but still gets a card at the end of the grid.

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
The project currently ships **9 `corelib-*` repos** (spanning **8 distinct languages** — Rust and C++ each have two builds) **+ a generator**. Two counts matter and they are **not** the same number:

- **library count = number of `corelib-*` repos** (currently 9) — used by the hero stat.
- **distinct-language count** (currently 8: C, C++, Rust, Go, Python, TypeScript, Java, C#) — used by the section heading and the JSON-LD array.

These appear in **five** places in `index.html`:

1. **Hero stat** — `<span><b>9</b> core libraries</span>` (the **library** count, = number of `corelib-*` repos).
2. **Section heading** — `<h2 class="sec-title">One format, eight languages</h2>` (the spelled-out **distinct-language** count).
3. **`.lang-grid`** — one `<a class="lang">` card per `corelib-*` repo + the generator card last. The card's `<small>` holds the **distinguishing blurb** from that repo's README intro (target / std vs no_std / speed vs size / runtimes) — this is how two builds of the same language are told apart. The repo slug lives only in the `href` now, not in the visible text.
4. **Footer columns** — `Core libraries` and `More languages` link lists (one `<a>` per corelib; label same-language variants distinctly, e.g. `Rust` vs `Rust (no_std)`).
5. **JSON-LD** — `"programmingLanguage": [ ... ]` in the `<script type="application/ld+json">` block. This is the list of **distinct languages**, so a second build of an existing language (e.g. a new Rust variant) does **not** add an entry.

…plus the **meta keywords** (`<meta name="keywords">`) and **meta description** mention languages generically — only touch if the framing changes.

> ⚠️ Keep the two counts straight: a new `corelib-*` repo **always** bumps the hero library count and adds a grid card + footer link, but it only bumps the heading number / adds a JSON-LD entry **if it introduces a brand-new language**. A second build of an existing language (the common case for embedded vs cloud) does not.

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

Confirm both counts from §3A are consistent: the **library** count (hero stat = number of `corelib-*` repos = grid cards minus the generator) and the **distinct-language** count (heading word = JSON-LD array length). Remember they differ whenever a language has more than one build.

---

## 6. Shipping changes

GitHub Pages publishes from the **`main` branch, root path**. So:

1. Branch off `main`, commit with a plain message (**no Co-Authored-By / AI trailer**).
2. Push and open a PR (`gh` or the API). The user supplies a token when needed — **never commit or hard-code a token**, and remind them to revoke it afterward.
3. **Merging the PR to `main` deploys to production** (`https://sofa-buffers.github.io`). Only merge when the user explicitly says so. After merge, Pages takes ~1–4 minutes to rebuild; verify the live site reflects the change (and suggest a hard refresh).

---

## 7. Quick recipe — "a new core library was added"

First read the new repo's README intro to write its `<small>` blurb, then branch on whether it's a new **language** or another **build of an existing one**.

**Case 1 — brand-new language** (say `corelib-kt`, Kotlin):

1. **Hero stat:** library count `9` → `10`.
2. **Heading:** "eight languages" → "nine languages" (spelled out).
3. **`.lang-grid`:** add a card (copy an existing `<a class="lang">`, pick a distinct `.badge` background color, set the repo URL + display name, and put the README-derived blurb in `<small>`).
4. **Footer:** add the link under `Core libraries` or `More languages`.
5. **JSON-LD:** add `"Kotlin"` to the `programmingLanguage` array.
6. **(Optional) meta keywords:** add the language if it helps SEO.
7. Validate (§5) → branch + PR (§6) → ask before merging.

**Case 2 — another build of an existing language** (say `corelib-rs-gpu`, a second Rust target):

1. **Hero stat:** library count `9` → `10`.
2. **Heading:** **no change** (no new language).
3. **`.lang-grid`:** add a card next to the sibling build; the badge/name repeat the language, and the `<small>` blurb is what distinguishes them (e.g. "GPU-offload Rust build" vs "High-speed std build").
4. **Footer:** add a distinctly-labelled link (e.g. `Rust (GPU)`).
5. **JSON-LD:** **no change** (language already listed).
6. Validate (§5) → branch + PR (§6) → ask before merging.

Removing or renaming a core library is the same checklist in reverse — and when removing one build of a multi-build language, drop the JSON-LD/heading entry **only** if it was the last build of that language.
