# 🎓 Handwritten Study-Guide Generator — Reusable Prompt

Copy everything in the **PROMPT** block below into a fresh AI coding session (opencode, Claude Code, Cursor, etc.), replace the two placeholders, and run it. The agent will turn any course/source folder into a colourful, handwritten-style study guide PDF that matches the existing *Docker & Kubernetes* guide.

Placeholders to fill in:
- `<<COURSE_NAME>>` — e.g. `Terraform in Depth`
- `<<SOURCE_PATH>>` — path to the course content (video transcripts, READMEs, docs, slides, a repo, or a URL)

---

## PROMPT

> **GOAL**
> Build a single, beautiful, print-ready **PDF study guide** for the course **`<<COURSE_NAME>>`**, sourced from **`<<SOURCE_PATH>>`**. It must look like vibrant, hand-written school notes (the existing *Docker & Kubernetes* guide is the visual reference) and be self-contained (embedded fonts, diagrams, cheat sheets, per-chapter takeaways, checklists and practice Q&A).
>
> **REUSE THE EXISTING TEMPLATE — DO NOT REINVENT THE STYLE**
> The repository already contains a complete, working pipeline under `study-guide/`. Treat it as a template:
> - `study-guide/styles.css` → the entire handwritten-notes design system + `@font-face` fonts. **Reuse as-is** (add new colour variants only if you need more than the built-in palette).
> - `study-guide/build.py` → two-pass HTML → PDF builder (Chrome headless render, then PyMuPDF page-number + bookmark stamping; sets PDF metadata author/title). **Reuse as-is.**
> - `study-guide/fonts/` → self-contained TTF/OTF webfonts (Caveat, Kalam, Nunito, JetBrains Mono). Reuse.
> - `study-guide/parts/` → one HTML file per chapter plus cover and TOC. This is the **content format** you will author.
> - The existing `study-guide/parts/*.html` files are worked examples of every component — copy their structure and class usage.
>
> **STEP 1 — INGEST THE COURSE**
> 1. Enumerate every section/chapter of the source (read all READMEs, transcripts, subtitles, slide text, or the docs). Preserve the author's ordering. If a section is large, split it into the smallest sensible chapters (like the Kubernetes "built-in resources" was split 4.1–4.7).
> 2. For each section, extract: the *why* (motivation), core concepts, exact commands/code/config, diagrams worth redrawing, gotchas, warnings, best practices, and any tables.
> 3. Note the number of chapters → this drives numbering and the Table of Contents.
>
> **STEP 2 — SET UP THE BUILD (only if the template files don't exist yet)**
> Create `study-guide/` with `styles.css`, `build.py`, `fonts/`, and an empty `parts/`. `build.py` must:
> - sort `parts/*.html` alphabetically and concatenate them between a `HEAD`/`FOOT` wrapper into `study-guide.html`;
> - **pass 1**: render with Chrome headless (`google-chrome --headless=new --no-sandbox --disable-gpu --no-pdf-header-footer --print-to-pdf=... file://...`) to discover page numbers from invisible `@@TOKEN@@` markers;
> - substitute `{{PAGE:TOKEN}}` placeholders in the TOC with real page numbers;
> - **pass 2**: render the final HTML → PDF;
> - stamp a footer with the page number (`Page N / TOTAL`) on every page **except the cover** (do not add any other running footer text);
> - build PDF bookmarks from each section's `data-toc`/`data-level` (decode HTML entities); set PDF metadata (title/author/subject).
> - Output the final PDF to the repo root as `<<COURSE_NAME>>-Study-Guide.pdf`.
>
> **STEP 3 — AUTHOR THE CONTENT**
> Create one `parts/NN-slug.html` per chapter (zero-padded so alphabetical sort = course order), plus:
> - `parts/00-cover.html` → `<section class="cover">` with a big handwritten title (two-word titles look best), an eyebrow line, a subtitle, feature badges, corner SVG doodles, an author credit line, and a footer note.
> - `parts/01-toc.html` → `<section class="chapter toc">` listing every chapter with `<span class="tpage">{{PAGE:TOKEN}}</span>`, grouped by Part with coloured `.part-title` headers.
> - Cheat sheets at the end (`parts/…-cheat-<topic>.html` and a final master cheat sheet).
>
> **Required structure of every chapter section:**
> ```html
> <section class="chapter acc-<colour>" data-toc="<N> · <Title>" data-level="2">
>   <span class="pgmark">@@<TOKEN>@@</span>
>   <div class="chapter-head">
>     <div class="eyebrowline">
>       <div class="chapter-num"><N></div>
>       <div>
>         <span class="kicker"><Course> · Part <N></span>
>         <h1><Title></h1>
>         <div class="chapter-sub"><one-line summary></div>
>       </div>
>     </div>
>     <div class="sepline"></div>
>   </div>
>   <!-- body: headings, boxes, grids, diagrams, tables, code, flow charts -->
>   <div class="takeaways t-<colour>"> … 🌟 Key Takeaways … </div>
>   <div class="checklist"> … ✅ Quick-Review Checklist … </div>
>   <div class="qa"> … 📝 Practice Questions (Q + ➜ A) … </div>
> </section>
> ```
> Every chapter MUST end with the three blocks: **Key Takeaways, Quick-Review Checklist, Practice Q&A** (4–6 questions each). Rotate `acc-`/`t-` colours so consecutive chapters differ.
>
> **Design-system components to use (defined in `styles.css`):**
> - Boxes: `.box` + `.def` (definition) / `.term` / `.formula` / `.example` / `.warning` / `.tip` / `.note` / `.remember`, each with a `.btitle` (emoji `.ico` + heading).
> - Layout: `.grid2` / `.grid3` / `.grid4` / `.grid4`, `.flow` with `.chip`/`.arw`, `.timeline` with `.tl-item`, `.diagram` (inline SVG), `.sticky` notes (`.s-yellow/.s-pink/.s-mint/.s-sky/.s-lilac/.s-peach`).
> - Tables: `<table class="tbl sky|mint|peach|pink|aqua|yellow">`.
> - Code: `<pre>` (terminal styling, comment span `<span class="c">`), `.codecap` captions, inline `<code>`.
> - Highlights: `.hl-pink/.hl-mint/.hl-sky/.hl-lilac`, `mark`, `.pill`, `.scribble`, `.rule`.
> - Accent palette: `sky, mint, lilac, peach, pink, aqua, yellow` (+ any you add, e.g. `indigo, amber, coral, green, teal, rose, red`).
> - Emojis in headings/titles are welcome and render correctly.
>
> **Content rules:**
> - Faithful to the source, but *rewritten* as concise revision notes — never dump raw transcripts.
> - Include the real, exact commands/code/YAML/config from the course (verified against the source files where available).
> - Add redrawn diagrams (inline SVG) for anything visual/architectural.
> - Flag pitfalls with `.warning`, pro-tips with `.tip`, and memory hooks with `.sticky` / `.scribble`.
> - Keep each chapter roughly 2–4 printed pages.
>
> **STEP 4 — BUILD & QA (iterate until clean, do this after every few chapters, not just at the end)**
> 1. Run `cd study-guide && python3 build.py`. Confirm it prints a page count and `Final PDF: …`.
> 2. Verify with a short PyMuPDF script:
>    - no unresolved `{{PAGE:…}}` or `@@TOKEN@@` text remains (except intentionally near-invisible `.pgmark`);
>    - the TOC page numbers are populated;
>    - bookmarks exist for every chapter with clean (entity-decoded) titles;
>    - **no content block overflows the page** (check text block bboxes against page rect minus a small margin; ignore the page-number footer band);
>    - fonts are embedded.
> 3. Do **not** try to "look" at rendered PNGs unless the model supports vision — rely on the programmatic bbox checks.
> 4. Keep the source content dirs untouched.
>
> **DELIVERABLE**
> A finished `<<COURSE_NAME>>-Study-Guide.pdf` with: Cover → Table of Contents (with page numbers) → all chapters → topic cheat sheet(s) → Master cheat sheet. Report the final page count and output path.

---

## Tips for the human

- **Add a chapter later:** drop a new `parts/NN-…html` file in the right position and re-run `python3 build.py` — page numbers and bookmarks update automatically.
- **Change colours/theme:** edit the CSS variables in `study-guide/styles.css` (`:root { … }`).
- **Rename the course:** only the cover/TOC text and the output filename reference the course name; everything else is generic.
- **Author name / metadata:** set it once in the cover part and in `build.py`'s `doc.set_metadata({...})`.
- **Fonts:** keep all fonts local in `study-guide/fonts/` so the PDF stays self-contained and offline-buildable.
