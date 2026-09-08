# WCAG Accessibility Remediation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix all 18 reported issues (15 Critical, 2 Serious, 1 Minor) plus the 1 unscored "best practice" failure from `wcgareport.pdf`, without changing the site's visual design.

**Architecture:** This is a static Jekyll site (GitHub Pages auto-build, no CI). The live site is built from `_layouts/*.html`, `index.md`, `_includes/*.md`, `_config.yml`, and `assets/css/*.css` — **not** from `html_source_file/` (a stale, separately-committed static export mentioned in `README.md` for non-Jekyll users; it is not what GitHub Pages serves and is out of scope) or `_site/` (gitignored build output). Every fix below targets the real Jekyll source. Fixes are grouped by root cause: most of the 18 reported failures trace back to only 7 source locations, because `_layouts/default.html` and `_layouts/simple.html` are both rendered per-page.

**Tech Stack:** Jekyll (`github-pages` gem), Liquid templates, plain CSS, vanilla JS (`vanilla-back-to-top.min.js`, jQuery already loaded).

**Spec:** `wcgareport.pdf` (AccessScan audit, 2026-09-08) — read in full during planning; findings summarized in the table below. No other spec doc exists for this work.

## Global Constraints

- Do not change the visual appearance/layout of the site (colors, fonts, positions) except where the fix *is* a layout bug (footer/topnav overlap).
- Do not hand-edit `html_source_file/` or `_site/` — they are not the deployed source.
- Do not hand-edit `assets/js/vanilla-back-to-top.min.js` (third-party minified vendor file) — patch its output via a small script instead.
- Every template change must be verified by running `bundle exec jekyll build --destination /tmp/_site_check` and grepping the generated HTML in `/tmp/_site_check`, since there is no automated test suite for this site.
- Match existing code style: the templates use unquoted/mixed HTML attribute style, inline `<style>`/`style=""` blocks, and custom non-semantic tags (`<autocolor>`, `<email>`, `<position>`) — leave those choices alone except where they are the direct cause of a reported failure.

---

## Findings → Fix Map

| # | Report finding (severity) | Root cause | Fix (Task) |
|---|---|---|---|
| 1 | Interactive elements not identifiable (Critical, score 0) | Empty `href=""` on 2 links (missing `affiliation_link`), avatar `<a>` with no `href`, `#back-to-top` is a plain `<div>` | Task 3, Task 7 |
| 2 | Sticky footer can overlap focused content (Serious) | `footer{position:fixed}` renders on top of full-width `<section>` on `simple.html` pages | Task 8 |
| 3 | Sticky header can overlap focused content (Serious) | Fixed `.topnav` height not reserved for in `section`/`header` top padding | Task 8 |
| 4 | Viewport disables zoom (Critical) | `user-scalable=no` in 3 layout files | Task 2 |
| 5 | Elements visually hidden but still exposed to AT (Critical, score 43) | Empty `<h1>`/`<h2>` in `index.md`, empty footer `<div><p><small>` when `imprint`/`data_protection` unset | Task 5, Task 6 |
| 6 | Image missing text alternative (Critical, score 50) | `<img src="ucsc.png">` has no `alt` | Task 3 |
| 7 | Link purpose unclear (Critical, score 67) | Icon-only social/CV/Scholar links have no accessible name | Task 4 |
| 8 | Decorative graphic clutter (Minor, score 80) | Same missing `alt` on `ucsc.png` as #6 | Task 3 |
| 9–17 | All scored 100 (Pass) | No action needed | — |
| 18–60 | "Relevant: No" | Not applicable to this site | — |
| Best practice #1 | Warn before PDF opens (Fail, unscored) | CV link has no indication it opens a PDF | Task 4 |

One report item is **not** mapped to a fix: item #5's failed-elements list includes a bare `<body></body>` snapshot with no attributes or surrounding context, which does not correspond to any static markup found in the templates (it may be a runtime/third-party-script artifact). Task 9 includes a step to investigate this live rather than guessing at a fix.

---

## File Structure

- Modify `assets/css/style.css` — add a `.sr-only` utility class (Task 1) and layout offset fixes (Task 8).
- Modify `assets/css/nav.css` — add `scroll-padding-top` (Task 8).
- Modify `_layouts/default.html` — viewport, logo/avatar links, social icons, footer, back-to-top (Tasks 2–4, 6, 7).
- Modify `_layouts/simple.html` — same as above minus avatar/heading concerns (Tasks 2–4, 6, 7, 8).
- Modify `404.html` — viewport only (Task 2, bonus consistency fix).
- Modify `_config.yml` — set `affiliation_link` (Task 3).
- Modify `index.md`, `_includes/talks.md`, `_includes/publications.md` — empty headings (Task 5).

---

### Task 1: Add a `.sr-only` visually-hidden utility class

**Files:**
- Modify: `assets/css/style.css`

**Interfaces:**
- Produces: CSS class `.sr-only`, used by Task 5 to give empty headings an accessible name without changing their visual appearance.

- [ ] **Step 1: Add the utility class**

Append to the end of `assets/css/style.css`:

```css
/* Visually hide content while keeping it available to screen readers */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

- [ ] **Step 2: Verify**

Run: `grep -n "sr-only" assets/css/style.css`
Expected: the new rule block is present.

- [ ] **Step 3: Commit**

```bash
git add assets/css/style.css
git commit -m "a11y: add sr-only utility class"
```

---

### Task 2: Allow pinch-zoom (remove `user-scalable=no`)

**Files:**
- Modify: `_layouts/default.html:7`
- Modify: `_layouts/simple.html:7`
- Modify: `404.html:6`

**Interfaces:** none (independent of other tasks).

- [ ] **Step 1: Fix `_layouts/default.html`**

Change:
```html
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
```
to:
```html
    <meta name="viewport" content="width=device-width, initial-scale=1" />
```

- [ ] **Step 2: Fix `_layouts/simple.html`** — same change, same line number.

- [ ] **Step 3: Fix `404.html`** — same change, same line number. (Note: `404.html` has no Liquid front matter, so Jekyll copies it verbatim — confirmed via `_site/404.html` still containing literal `{{ site.favicon_dark }}`. This edit is a plain string replacement, unrelated to that pre-existing bug, which is out of scope.)

- [ ] **Step 4: Verify**

Run:
```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -rn "user-scalable" /tmp/_site_check/ || echo "CLEAN: no user-scalable left"
```
Expected: `CLEAN: no user-scalable left`

- [ ] **Step 5: Commit**

```bash
git add _layouts/default.html _layouts/simple.html 404.html
git commit -m "a11y: allow pinch-zoom by removing user-scalable=no"
```

---

### Task 3: Fix empty/missing link destinations and missing image `alt`

**Files:**
- Modify: `_config.yml`
- Modify: `_layouts/default.html:63,99,109`
- Modify: `_layouts/simple.html:86`

**Interfaces:** none.

- [ ] **Step 1: Set a real affiliation link in `_config.yml`**

`affiliation_link` is currently commented out, which is why every `{{ site.affiliation_link }}` render produces `href=""`. Change:
```yaml
# affiliation_link: https://www.illinois.edu/
```
to:
```yaml
affiliation_link: https://www.ucsc.edu/
```

- [ ] **Step 2: Add `alt` text to the institution logo in `_layouts/default.html`**

Change line 63:
```html
      <a href="{{ site.affiliation_link }}"><img width="40" src="../{{ site.affiliation_logo }}"></a>
```
to:
```html
      <a href="{{ site.affiliation_link }}"><img width="40" src="../{{ site.affiliation_logo }}" alt="{{ site.affiliation }} logo" /></a>
```
This resolves report items #1 (interactive element without a real name — now has both a real `href` and an `alt`-bearing image), #6 and #8 (missing/decorative image alt).

- [ ] **Step 3: Fix the avatar link in `_layouts/default.html`**

Line 99 currently has an `<a>` with **no `href` attribute at all**, so it isn't a functioning link — it exists only to visually wrap the image, but browsers/AT may still expose it as an empty interactive element. Since the adjacent `<h1>{{ site.title }}</h1>` (line 102) already states the name, the avatar image is redundant/decorative. Remove the dead wrapper and mark the image decorative:

Change:
```html
        <a class="image avatar"><img src="{{ site.baseurl }}{{ site.avatar }}" alt="avatar" /></a>
```
to:
```html
        <span class="image avatar"><img src="{{ site.baseurl }}{{ site.avatar }}" alt="" /></span>
```
(`<span>` keeps the existing `.image.avatar` CSS selector working unchanged; `alt=""` marks the image as decorative per WCAG since its content is redundant with the following heading.)

- [ ] **Step 4: Verify affiliation link renders correctly in `_layouts/default.html` line 109**

No template change needed here — it already reads `<a href="{{ site.affiliation_link }}" rel="noopener"><autocolor>{{ site.affiliation }}</autocolor></a>`; Step 1's config change fixes its `href` automatically.

- [ ] **Step 5: Apply the same logo `alt` fix to `_layouts/simple.html`**

Change line 86:
```html
      <a href="{{ site.affiliation_link }}"><img width="120" src="../{{ site.affiliation_logo }}"></a>
```
to:
```html
      <a href="{{ site.affiliation_link }}"><img width="120" src="../{{ site.affiliation_logo }}" alt="{{ site.affiliation }} logo" /></a>
```

- [ ] **Step 6: Verify**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n 'href=""' /tmp/_site_check/index.html /tmp/_site_check/services/index.html
grep -n '<img width="40"' /tmp/_site_check/index.html
```
Expected: no `href=""` matches; the `<img width="40">` line now includes `alt="University of California, Santa Cruz logo"`.

- [ ] **Step 7: Commit**

```bash
git add _config.yml _layouts/default.html _layouts/simple.html
git commit -m "a11y: fix empty link hrefs and add missing image alt text"
```

---

### Task 4: Give icon-only links an accessible name

**Files:**
- Modify: `_layouts/default.html:130-160`
- Modify: `_layouts/simple.html` (topnav hamburger icon at line 92-94; no social-icons block exists there)

**Interfaces:** none.

- [ ] **Step 1: Add `aria-label` to each social/Scholar/CV icon link and hide the decorative glyph from AT**

In `_layouts/default.html`, replace the `social-icons` block (lines 131-161):

```html
        <div class="social-icons">
        {% if site.google_scholar %}
        <a style="margin: 0 5px 0 0" href="{{ site.google_scholar }}">
          <i class="ai ai-google-scholar" style="font-size:1.2rem"></i>
        </a>  
        {% endif %}

        {% if site.cv_link %}
        <a style="margin: 0 5px 0 0" href="{{ site.cv_link }}">
          <i class="ai ai-cv" style="font-size:1.3rem;"></i>
        </a>
        {% endif %}

        {% if site.github_link %}
        <a style="margin: 0 5px 0 0" href="{{ site.github_link }}">
          <i class="fab fa-github"></i>
        </a>
        {% endif %}

        {% if site.linkedin %}
        <a style="margin: 0 5px 0 0" href="{{ site.linkedin }}">
          <i class="fab fa-linkedin"></i>
        </a>
        {% endif %}

        {% if site.twitter %}
        <a style="margin: 0 0 0 0" href="{{ site.twitter }}">
          <i class="fab fa-x-twitter"></i>
        </a>
        {% endif %}
        </div>
```

with:

```html
        <div class="social-icons">
        {% if site.google_scholar %}
        <a style="margin: 0 5px 0 0" href="{{ site.google_scholar }}" aria-label="Google Scholar profile">
          <i class="ai ai-google-scholar" style="font-size:1.2rem" aria-hidden="true"></i>
        </a>  
        {% endif %}

        {% if site.cv_link %}
        <a style="margin: 0 5px 0 0" href="{{ site.cv_link }}" aria-label="Download curriculum vitae (opens PDF)">
          <i class="ai ai-cv" style="font-size:1.3rem;" aria-hidden="true"></i>
        </a>
        {% endif %}

        {% if site.github_link %}
        <a style="margin: 0 5px 0 0" href="{{ site.github_link }}" aria-label="GitHub profile">
          <i class="fab fa-github" aria-hidden="true"></i>
        </a>
        {% endif %}

        {% if site.linkedin %}
        <a style="margin: 0 5px 0 0" href="{{ site.linkedin }}" aria-label="LinkedIn profile">
          <i class="fab fa-linkedin" aria-hidden="true"></i>
        </a>
        {% endif %}

        {% if site.twitter %}
        <a style="margin: 0 0 0 0" href="{{ site.twitter }}" aria-label="X (formerly Twitter) profile">
          <i class="fab fa-x-twitter" aria-hidden="true"></i>
        </a>
        {% endif %}
        </div>
```

This resolves report item #7 (link purpose unclear — all 4 failing icon links now have names) and the unscored best-practice PDF-warning failure (the CV link's label now says "opens PDF").

- [ ] **Step 2: Label the hamburger menu icon in both layouts**

`_layouts/default.html` line 69-71 and `_layouts/simple.html` line 92-94 both have:
```html
      <a href="javascript:void(0);" class="icon" onclick="myFunction();reverseLinks()">
        <i class="fa fa-bars"></i>
      </a>
```
Change both occurrences to:
```html
      <a href="javascript:void(0);" class="icon" onclick="myFunction();reverseLinks()" aria-label="Toggle navigation menu">
        <i class="fa fa-bars" aria-hidden="true"></i>
      </a>
```
(This wasn't a scored failure — the scanner marked this element "successful" already per item #5's success list — but it is the same defect class and free to fix consistently while editing this block; skip if you want to strictly minimize diff.)

- [ ] **Step 3: Verify**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n 'aria-label="Google Scholar profile"\|aria-label="Download curriculum vitae' /tmp/_site_check/index.html
```
Expected: both lines found.

- [ ] **Step 4: Commit**

```bash
git add _layouts/default.html _layouts/simple.html
git commit -m "a11y: add accessible names to icon-only links"
```

---

### Task 5: Fix empty headings

**Files:**
- Modify: `index.md:6,8`
- Modify: `_includes/talks.md:1`
- Modify: `_includes/publications.md:1`

**Interfaces:**
- Consumes: `.sr-only` class from Task 1.

- [ ] **Step 1: Give the `about-me` anchor a real accessible name**

In `index.md`, change:
```html
<h1 id="about-me"></h1>
```
to:
```html
<h1 id="about-me"><span class="sr-only">About Me</span></h1>
```

- [ ] **Step 2: Remove the purely decorative empty `<h2>`**

The second heading in `index.md` (`<h2 style="margin: 80px 0px 10px;"></h2>`) has no `id` and no text — it exists only to add vertical spacing before the bio paragraph, not as a real heading or anchor target. Replace it with a non-semantic spacer so it stops being announced as an empty heading:

Change:
```html
<h2 style="margin: 80px 0px 10px;"></h2>
```
to:
```html
<div style="margin-top: 80px;"></div>
```

- [ ] **Step 3: Fix `_includes/talks.md`**

Change:
```html
<h1 id="invited-talks"></h1>
```
to:
```html
<h1 id="invited-talks"><span class="sr-only">Invited Talks</span></h1>
```

- [ ] **Step 4: Fix `_includes/publications.md`**

Change:
```html
<h1 id="publications"></h1>
```
to:
```html
<h1 id="publications"><span class="sr-only">Publications</span></h1>
```

- [ ] **Step 5: Verify**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n '<h1 id="about-me">\|<h1 id="invited-talks">\|<h1 id="publications">' /tmp/_site_check/index.html /tmp/_site_check/talks/index.html /tmp/_site_check/publications/index.html
grep -c '<h1 id="about-me"></h1>\|<h2 style="margin: 80px' /tmp/_site_check/index.html
```
Expected: the first grep shows each heading now wraps a `sr-only` span; the second grep returns `0` (no bare empty headings remain).

- [ ] **Step 6: Commit**

```bash
git add index.md _includes/talks.md _includes/publications.md
git commit -m "a11y: give empty headings real accessible names"
```

---

### Task 6: Stop rendering an empty footer block when imprint/data-protection links are unset

**Files:**
- Modify: `_layouts/default.html:173-188`
- Modify: `_layouts/simple.html:140-155`

**Interfaces:** none.

- [ ] **Step 1: Wrap the footer content in an outer conditional in `_layouts/default.html`**

`site.imprint` and `site.data_protection` are not set in `_config.yml`, so today the footer always renders an empty `<div><p><small></small></p></div>` — this is report item #5's `<div style="font: 12px/1.2 Crimson Pro, serif">...` failing snapshot. Change:

```html
      <footer>
        <div style="font: 12px/1.2 Crimson Pro, serif">
        <p>
        <small>
        {% if site.imprint %}
        <a href="{{ site.imprint }}" rel="noopener"><autocolor>Imprint / Impressum</autocolor></a><br>
        {% endif %}
        {% if site.data_protection %}
        <a href="{{ site.data_protection }}" rel="noopener"><autocolor>Data Protection / Datenschutzhinweis</autocolor></a>
        {% endif %}
        </small>
        </p>
        </div>
        <br>
        <br>
      </footer>
```

to:

```html
      <footer>
        {% if site.imprint or site.data_protection %}
        <div style="font: 12px/1.2 Crimson Pro, serif">
        <p>
        <small>
        {% if site.imprint %}
        <a href="{{ site.imprint }}" rel="noopener"><autocolor>Imprint / Impressum</autocolor></a><br>
        {% endif %}
        {% if site.data_protection %}
        <a href="{{ site.data_protection }}" rel="noopener"><autocolor>Data Protection / Datenschutzhinweis</autocolor></a>
        {% endif %}
        </small>
        </p>
        </div>
        {% endif %}
        <br>
        <br>
      </footer>
```

- [ ] **Step 2: Apply the identical change to `_layouts/simple.html`** (same block, lines 141-152).

- [ ] **Step 3: Verify**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n '<small></small>\|<p>\s*</p>' /tmp/_site_check/index.html /tmp/_site_check/services/index.html
```
Expected: no matches (the empty `<p><small></small></p>` is gone from the built HTML).

- [ ] **Step 4: Commit**

```bash
git add _layouts/default.html _layouts/simple.html
git commit -m "a11y: don't render empty footer markup when no legal links are configured"
```

---

### Task 7: Make the "Back to Top" control keyboard- and screen-reader-accessible

**Files:**
- Modify: `_layouts/default.html:76-93`
- Modify: `_layouts/simple.html:99-116`

**Interfaces:** none.

`vanilla-back-to-top.min.js` (a vendor file we must not hand-edit — see Global Constraints) injects a plain `<div id="back-to-top" class="hidden">Back to Top</div>` with a click handler but no `role`, no `tabindex`, and no keyboard handler — exactly report item #1's second failing snapshot. We patch its output after it renders.

- [ ] **Step 1: Patch the injected element in `_layouts/default.html`**

Change:
```html
    <script src="../assets/js/vanilla-back-to-top.min.js"></script>
    <script>addBackToTop({
        backgroundColor: '#fff',
        innerHTML: 'Back to Top',
        textColor: '#333'
      })
    </script>
```
to:
```html
    <script src="../assets/js/vanilla-back-to-top.min.js"></script>
    <script>addBackToTop({
        backgroundColor: '#fff',
        innerHTML: 'Back to Top',
        textColor: '#333'
      })
      document.addEventListener('DOMContentLoaded', function () {
        var backToTop = document.getElementById('back-to-top');
        if (!backToTop) return;
        backToTop.setAttribute('role', 'button');
        backToTop.setAttribute('tabindex', '0');
        backToTop.setAttribute('aria-label', 'Back to top');
        backToTop.addEventListener('keydown', function (event) {
          if (event.key === 'Enter' || event.key === ' ') {
            event.preventDefault();
            backToTop.click();
          }
        });
      });
    </script>
```

- [ ] **Step 2: Apply the identical change to `_layouts/simple.html`** (same block, lines 100-105).

- [ ] **Step 3: Verify**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n "role', 'button'" /tmp/_site_check/index.html /tmp/_site_check/services/index.html
```
Expected: both files contain the patch script.

Manual check (can't be grepped): serve the site (`bundle exec jekyll serve`), open in a browser, Tab to the "Back to Top" control, confirm it receives visible focus and that pressing Enter/Space scrolls to top.

- [ ] **Step 4: Commit**

```bash
git add _layouts/default.html _layouts/simple.html
git commit -m "a11y: make Back to Top control keyboard- and screen-reader-accessible"
```

---

### Task 8: Fix fixed header/footer/topnav overlapping page content

**Files:**
- Modify: `assets/css/nav.css`
- Modify: `assets/css/style.css`
- Modify: `_layouts/simple.html:119` (the inline `style="float: none; width: auto;"` on `<section>`)

**Interfaces:** none.

Two distinct real overlaps cause report items #2 and #3:

1. **`.topnav`** (`position: fixed; width: 100%; z-index: 999`) sits at the very top of *every* page. `section`'s own top padding (`padding-top: 1.2em` ≈ 19px, `assets/css/style.css:125`) is far smaller than the topnav's rendered height (~60-70px from its logo image + nav-link row), so the fixed bar can visually sit on top of the first line(s) of page content, obscuring a focused element there.
2. On **`simple.html`** pages (services, publications, talks, experience), `<section>` is given `float: none; width: auto`, so it spans the *entire* 960px wrapper — but `footer` (`position: fixed; float: left; width: 232px; bottom: 30px`, `assets/css/style.css:131`) still renders pinned over the wrapper's left 232px column, on top of that full-width section, potentially hiding focused links near the page's bottom-left.

- [ ] **Step 1: Reserve space for the fixed topnav via `scroll-padding-top`**

Add to the top of `assets/css/nav.css`:
```css
/* Keep in-page jumps and focused elements clear of the fixed top nav bar */
html {
  scroll-padding-top: 90px;
}
```

- [ ] **Step 2: Increase `section`'s top padding in `assets/css/style.css`**

Change:
```css
section { width: 650px; float: right; padding-top: 1.2em; padding-bottom: 50px; }
```
to:
```css
section { width: 650px; float: right; padding-top: 90px; padding-bottom: 50px; }
```

- [ ] **Step 3: Give the full-width `simple.html` section a left offset so it clears the fixed footer column**

In `assets/css/style.css`, add a new rule right after the `section` rule from Step 2:
```css
section.full-width { float: none; width: auto; padding-left: 250px; }

@media print, screen and (max-width: 960px) {
  section.full-width { padding-left: 0; }
}
```
(The existing `@media (max-width: 960px) { header, section, footer { float: none; position: static; width: auto; } }` rule at `assets/css/style.css:133` already resets `float`/`position`/`width` at that breakpoint, so the extra rule above only needs to reset the new `padding-left`.)

In `_layouts/simple.html`, change:
```html
      <section style="float: none; width: auto;">
```
to:
```html
      <section class="full-width">
```

- [ ] **Step 4: Verify with a build + visual check**

```bash
bundle exec jekyll build --destination /tmp/_site_check
grep -n "scroll-padding-top" assets/css/nav.css
grep -n "padding-top: 90px" assets/css/style.css
grep -n 'class="full-width"' /tmp/_site_check/services/index.html
```
Expected: all three found.

Then visually confirm no overlap using headless Chrome (already installed at `/usr/bin/google-chrome`):
```bash
bundle exec jekyll serve --destination /tmp/_site_check --detach --port 4001
google-chrome --headless --disable-gpu --window-size=1280,1024 --screenshot=/tmp/home.png http://localhost:4001/
google-chrome --headless --disable-gpu --window-size=1280,1024 --screenshot=/tmp/services.png http://localhost:4001/services/
```
Open `/tmp/home.png` and `/tmp/services.png` and confirm: (a) the topnav does not cover the page title/first content line, (b) the footer does not cover any body text on the services page. Stop the server afterward (`kill $(cat /tmp/_site_check/.jekyll-*.pid 2>/dev/null)` or find the `jekyll serve` process and kill it).

- [ ] **Step 5: Commit**

```bash
git add assets/css/nav.css assets/css/style.css _layouts/simple.html
git commit -m "a11y: stop fixed topnav/footer from overlapping page content"
```

---

### Task 9: Full-site verification and investigation of the unexplained `<body></body>` finding

**Files:** none modified; verification only.

- [ ] **Step 1: Rebuild the whole site clean**

```bash
rm -rf /tmp/_site_check
bundle exec jekyll build --destination /tmp/_site_check
```
Expected: build succeeds with no errors.

- [ ] **Step 2: Re-check every fix at once**

```bash
grep -rl "user-scalable" /tmp/_site_check/ | grep -v html_source_file; echo "^ should be empty"
grep -rn 'href=""' /tmp/_site_check/index.html /tmp/_site_check/services/index.html /tmp/_site_check/talks/index.html /tmp/_site_check/publications/index.html /tmp/_site_check/teaching/index.html
grep -c 'aria-label="Google Scholar profile"' /tmp/_site_check/index.html
grep -c '<h1 id="about-me"><span class="sr-only">' /tmp/_site_check/index.html
grep -c "role', 'button'" /tmp/_site_check/index.html
```
Expected: first grep empty; `href=""` greps empty; all `-c` counts are `1` or more.

- [ ] **Step 3: Investigate the unmapped `<body></body>` finding**

Re-open `wcgareport.pdf` page 5 (item #5's failed-elements list, entry 4) and re-run the scan (or an equivalent tool such as axe DevTools / Lighthouse) against the rebuilt site once it's deployed, since this snapshot didn't match any static template markup found during planning — it may be produced at runtime by one of the third-party scripts (`github-stars.js`, `favicon-switcher.js`, `scale.fix.js`) rather than existing in the source. If it reproduces, capture the actual DOM node (via browser DevTools "Inspect" on the flagged element) and file it as a follow-up fix; if it doesn't reproduce, note it as resolved by the Task 6 footer fix (most likely candidate, since removing the empty footer `<div>` may have been what the scanner's snapshot was clipping).

- [ ] **Step 4: Re-run the original scanner (or equivalent) against the deployed site**

After merging and GitHub Pages redeploys, re-run whatever tool produced `wcgareport.pdf` (or a substitute such as https://www.deque.com/axe/devtools/ or Lighthouse's Accessibility audit in Chrome DevTools) against the live URL and confirm the issue count drops from 18 to 0 (or document any remaining items with justification).

- [ ] **Step 5: Update the plan doc with results**

Add a `## Results` section to the bottom of this file summarizing the before/after scan, then commit:
```bash
git add docs/superpowers/plans/2026-09-08-wcag-accessibility-fixes.md
git commit -m "docs: record WCAG remediation verification results"
```

## Results

All 9 tasks completed on branch `a11y/wcag-remediation`, executed via superpowers:subagent-driven-development (fresh implementer + reviewer subagent per task). One fix round was needed (Task 2); every other task's review came back clean on the first pass.

**Deviations from the plan as originally written**, both discovered during implementation, not before:

1. **Task 2 fix round**: the implementer edited 6 files under `html_source_file/` in addition to the 3 authorized files, to make an overly broad self-chosen verification grep pass. `html_source_file/` is a stale static export (see Architecture section above) explicitly out of scope. Reverted via `git checkout <pre-task-commit> -- <6 files>` + `git commit --amend`; scoped re-review confirmed clean.
2. **Task 8 root-cause correction**: the plan's Task 8 Step 3 (a `padding-left: 250px` / `.full-width` CSS class applied to `_layouts/simple.html`, to fix a fixed-footer-over-content overlap on "services/publications/talks/experience" pages) was written on a mistaken assumption. Grepping every `.md` source file confirmed those pages all declare `layout: default`, and nothing in the repo uses `layout: simple` — that layout is dead code. Dropped that part of the task entirely (kept the `scroll-padding-top`/`section padding-top` fix, which the implementer's own before/after screenshots confirmed genuinely fixes the real topnav-overlap on `default.html`, the layout every page actually uses).
3. **New finding during Task 9 verification** (not in the original plan): placing this plan doc under `docs/superpowers/plans/` and leaving `wcgareport.pdf` at the repo root caused Jekyll to build both into the live site (the plan `.md` was even rendered to a public `.html` page) — `_config.yml`'s `exclude:` list didn't cover them. Fixed by adding both to that list (commit `ff6994d`).

**The one report item left unresolved**: item #5's `<body></body>` failed-element snapshot. Investigated (see the SDD ledger for Task 9) — the same report item's successful-elements list shows a hidden, inert iframe (`srcdoc="<html lang=\"en\"></html>"`, `visibility:hidden;pointer-events:none`) that isn't created by any script in this repo or any of its CDN dependencies. Most likely explanation: an artifact of the AccessScan tool's own scanning apparatus (or a browser extension active during the scan), not this site's markup. **Needs a live re-scan to confirm** — see follow-up below.

**Human-in-the-loop visual review (Task 8)**: per explicit instruction, the one task that changes visual layout was applied and verified (build + headless-Chrome before/after screenshots) but held uncommitted. The human partner reviewed it live via a local `bundle exec jekyll serve` on their own machine and approved it ("ok, move on, no visual problems") before it was committed.

**Verification performed:**
- Every task's diff went through a fresh-subagent task review (spec compliance + code quality); Task 2's fix round got a scoped re-review.
- A final clean `bundle exec jekyll build` plus targeted greps confirmed all 8 fix categories present in the built output.
- Task 7's reviewer-flagged runtime-behavior gaps (does the patched `#back-to-top` element actually receive the ARIA attributes at runtime, not just in source) were closed by the controller directly: built + served the site and used `google-chrome --headless --dump-dom` to inspect the POST-JS-EXECUTION DOM, confirming `role="button" tabindex="0" aria-label="Back to top"` are actually applied by the patch script.

**Follow-up for the human partner, outside this session's reach:**
- Re-run the original scanner (or axe DevTools / Lighthouse) against the deployed site once this branch is merged and GitHub Pages redeploys. Confirm the issue count drops from 18 (+1 unscored) to 0, and specifically check whether the `<body></body>`/hidden-iframe finding reproduces — if it does, it needs live DevTools inspection to identify its source, since nothing in this repo's own code produces it.
