---
name: generate-docs
description: Generate project documentation or research documentation as a self-contained HTML file (system/light/dark theme, jump-to-section nav menu, no external network calls), a plain Markdown file, a PDF, or any mix of the three — HTML + Markdown by default. Use this whenever the user asks to document, write up, explain, or create project documentation, research documentation, an overview/comparison/decision doc for a technical topic, system, or architecture choice, or a step-by-step guide (how-to, setup or install steps, runbook, with fill-in fields for values the reader supplies) — even if they just say "write this up," "document this," or "create an overview of X" without naming a format. Especially relevant for cross-project research, architecture explainers, build-vs-buy comparisons, and anything meant to be read later by the user or a colleague rather than just answered in chat.
---

# Generate docs

Produces a doc for a technical topic in up to four formats, all from the same source material — write the content once, then render it once per format:

- **HTML** — one self-contained `.html` file people can just double-click open.
- **Markdown** — the same content in a plain `.md` file (for pasting into a wiki, diffing in git, or reading in a terminal).
- **PDF** — a print of the HTML, for attaching, emailing or printing.
- **Claude artifact** — the same HTML published as a private page at a claude.ai link, which the user can share from claude.ai. Only offered when this session has the artifact publishing tools (`publish_files` / `publish_version`).

## Pick the formats first

Before writing anything, settle which formats to produce:

- **If the request already says** ("just the HTML", "markdown only", "and a PDF", "as an artifact", "give me a link"), use that and don't ask.
- **Otherwise ask once**, as a multi-select question with HTML, Markdown and PDF as the choices (plus Claude artifact when the publishing tools are available) and HTML + Markdown preselected / marked recommended. Any non-empty combination is valid (HTML only, Markdown only, HTML + PDF, artifact only…).
- **If you can't ask** (a non-interactive or scheduled run), produce HTML + Markdown.

## Why these shapes

The HTML version is the one people actually read: it's visual, it follows the reader's system light/dark setting (with a toggle to override it), and it has a jump-to-section menu so a 10-section doc doesn't turn into endless scrolling. The Markdown version exists because not every place a doc ends up (a git repo, a work wiki, a Slack paste) wants to run arbitrary HTML — Markdown travels better there. The PDF is for places that want a fixed, printable file (an email attachment, a ticket, a printout); it loses the interactivity, so it's an extra, not a replacement. The artifact is for sharing a link instead of a file: colleagues open it in claude.ai with the theme, nav and fields still working. HTML + Markdown is the default because together they cover reading and reuse.

## Before writing: gather the real content first

Don't start from the template. Start from the actual facts, numbers, trade-offs, and sources for the topic — research it, read the relevant code, or use what's already been established in the conversation. The template below is just the container; a good-looking doc with thin or made-up content is worse than no doc. If the topic involves comparing options (stack choices, tools, architectures), verify claims rather than reciting priors — e.g. check an actual LICENSE file before stating a tool is free, or check version/config files before describing what's "currently installed."

## HTML structure

Build the HTML whenever HTML, PDF or Claude artifact was chosen (the PDF is printed from it and the artifact is published from it; if only PDF was chosen, build the HTML in a scratch location and don't deliver it). One HTML file serves all three: don't fork a separate artifact version. Build one `<html>` file, inline `<style>` and `<script data-doc>` (always mark the doc's own scripts `data-doc`), nothing loaded from a CDN or network — it has to open and work offline, forever, with no dependency rot. Rough shape for explainers and comparisons, adapt section count/order to the topic (step-by-step guides use the outline under **Guides and fill-in fields** instead):

1. **Header** — title, one-line description, maybe a short stats row if there are a few headline numbers worth surfacing immediately.
2. **Why this / the problem** — a short plain-language framing before diving into mechanics. If there's a real-world analogy that helps (e.g. "think of the broker as the front desk for a building with many tenant companies"), use it.
3. **Core explainer section(s)** — the main content: what the thing is, how it works, comparisons. Prefer:
   - **Real `<table>` markup** for comparisons, not prose trying to do a table's job.
   - **CSS-grid or flexbox diagrams** for architecture/flow — boxes and arrows built from styled `<div>`s, themed with the same CSS variables as the rest of the page, rather than an external image. Inline SVG is also fine. Avoid pasting in a screenshot or an image that won't theme or scale.
   - **Scenario walkthroughs** ("what happens when X fails," "one request step by step") when tracing a concrete path through the system clarifies it better than describing it abstractly.
4. **Decision framework / verdict**, if the doc is a comparison — be concrete about when each option actually wins, not just a feature list. A reader should be able to find themselves in it ("if you have fewer than N servers, do A; past that, B").
5. **Open questions / limits** — say plainly what isn't decided or what the doc doesn't cover. Don't paper over gaps.
6. **References** — this section is always included, not optional. Link to the primary sources actually used while researching (official docs, license files, RFCs, vendor pages), plus anything the reader would need to download or install to act on the doc. Group them if there are several categories (e.g. "official docs," "libraries mentioned," "alternatives considered"). If a tool has lookalike/scam/reseller domains that show up in search results, name the real official one explicitly and call out the lookalikes by domain so the reader doesn't land on the wrong site. Every factual claim or comparison point in the doc should trace back to something in this list — don't cite a source in the body and then leave it out of References, and don't put a link in References that nothing in the doc actually relied on.

### Light/dark theme

**Default to the system theme**: with no saved choice, the page follows the reader's OS/browser light/dark setting (`prefers-color-scheme`) and switches live if that setting changes. A small toggle button in the header cycles **System → Light → Dark** and shows the current mode ("Theme: System"). Persist an explicit Light or Dark choice with `localStorage` (keyed per doc, like the fields' `STORE`); choosing System removes the saved key. Wrap every `localStorage` call in `try/catch` (the snippets below do): it works in a local file and usually in a published artifact, but a sandboxed frame may refuse it, and the page must still work — it just forgets the choice. Only start in a fixed theme if the user asks for one.

Use CSS custom properties on `:root` for the light palette, overridden under `[data-theme="dark"]`. The script always sets `data-theme` to the *resolved* theme, so the CSS needs only that one dark selector. Put the first part in a `<script>` at the top of `<head>` so the page never flashes the wrong theme:

```css
:root{color-scheme:light; /* light palette vars */}
[data-theme="dark"]{color-scheme:dark; /* dark palette vars */}
```

```html
<!-- top of <head>; every script the doc writes is marked data-doc (see guide-fields.md) -->
<script data-doc>
  var THEME_KEY = 'topic-name-theme';
  var darkMQ = window.matchMedia('(prefers-color-scheme: dark)');
  function savedTheme() { try { return localStorage.getItem(THEME_KEY); } catch (e) { return null; } } // 'light' | 'dark' | null (= system)
  function applyTheme() { var t = savedTheme(); document.documentElement.setAttribute('data-theme', t || (darkMQ.matches ? 'dark' : 'light')); }
  applyTheme();
</script>
```

```html
<button type="button" class="theme-btn" id="theme-btn"></button>
```

```js
var themeBtn = document.getElementById('theme-btn');
function labelTheme() { var t = savedTheme(); themeBtn.textContent = 'Theme: ' + (t === 'light' ? 'Light' : t === 'dark' ? 'Dark' : 'System'); }
themeBtn.addEventListener('click', function () {
  var next = { '': 'light', light: 'dark', dark: '' }[savedTheme() || ''];
  try { if (next) localStorage.setItem(THEME_KEY, next); else localStorage.removeItem(THEME_KEY); } catch (e) {}
  applyTheme(); labelTheme();
});
darkMQ.addEventListener('change', applyTheme); // only changes anything while on System
labelTheme();
```

### Jump-to-section menu

Every doc gets a navigation menu linking to each major section. This is what "context menu" or "nav menu" requests are almost always asking for, as distinct from a literal right-click menu — check with the user once if a request is ambiguous, but default to this reading.

- **Wide screens (≥ 1024px): a left pane, always open.** The page is a two-column grid: a sticky section list on the left, the doc on the right. Desktop readers have the width, so don't hide the sections behind a button — a collapsed floating button in a corner is the wrong default here.
- **Narrow screens (< 1024px): the same list collapses** into a sticky bar at the top with a "Sections" button that drops the list down. It's one `<nav>` restyled by a media query, not two copies of the list.
- **Highlight where the reader is** as they scroll: the section in view, and the part in view when the nav lists parts (below). Keep the highlighted link visible if the pane has to scroll.
- **One nav only.** Don't also put a row of section links in the header; the pane replaces it.

#### Subheadings in the nav

Sections often have parts (`<h3>`s). Decide section by section whether those parts get their own links, nested under the section's link:

- **List the parts** when a section has two or more that a reader could want to jump to on their own: separate components, interfaces, options, steps, scenarios, sub-analyses. Long, multi-part sections are where readers get lost, so these benefit most.
- **Keep it one link** when the section has a single `<h3>`, or its `<h3>`s are labels rather than destinations: card or panel titles in a side-by-side grid, or a few short groups under one heading (like vendor groups in References).
- **Two levels at most.** Never put `<h4>`s in the nav. A section that seems to need a third level is usually two sections.
- **Long nav, collapse the parts.** If the nav would have more than about 25 links in total, add the `collapse-subs` class to the `<nav>`: each section's parts then show only while the reader is in that section. Under about 25, show them all.
- Every listed `<h3>` needs an `id`. While one of its parts is active, the section's link stays marked in a quieter style, so the reader still sees which section they're in.
- The Markdown file needs nothing extra: the parts are already its `###` headings.

A starting point that meets all of this (adapt the colors to the page's own variables, keep the behavior):

```html
<div class="layout">
  <nav class="sidenav" id="sidenav" aria-label="On this page"> <!-- add class "collapse-subs" past ~25 links -->
    <button type="button" class="sidenav-btn" aria-expanded="false" aria-controls="sidenav-list">Sections</button>
    <p class="sidenav-title">On this page</p>
    <ol id="sidenav-list">
      <li><a href="#problem">The problem</a></li>
      <li><a href="#interfaces">Interfaces</a>
        <ol>
          <li><a href="#agent-api">Agent HTTP API</a></li>
          <li><a href="#app-api">App API</a></li>
        </ol>
      </li>
      <!-- one <li> per section, in page order -->
    </ol>
  </nav>
  <div class="content"><!-- header, main, footer --></div>
</div>
```

```css
.sidenav ol{list-style:none; margin:0; padding:0}
.sidenav a{display:block; padding:5px 12px; border-left:2px solid transparent; color:var(--muted); text-decoration:none; font-size:.9rem; line-height:1.35}
.sidenav a:hover{color:var(--fg)}
.sidenav a[aria-current="true"]{color:var(--accent); border-left-color:var(--accent); font-weight:650}
/* parts: indented under their section, which stays marked while one of its parts is active */
.sidenav ol ol{margin:0 0 4px}
.sidenav ol ol a{padding:3px 12px 3px 26px; font-size:.84rem}
.sidenav li.in-path > a:not([aria-current="true"]){color:var(--fg); font-weight:650}
.sidenav.collapse-subs li:not(.in-path) > ol{display:none}
/* narrow: sticky top bar + drop-down */
.sidenav{position:sticky; top:0; z-index:20; margin:-32px -16px 24px; padding:8px 16px; background:var(--bg); border-bottom:1px solid var(--line)}
.sidenav-title{display:none}
.sidenav-btn{font:inherit; font-size:.9rem; font-weight:650; padding:6px 14px; border-radius:999px; border:1px solid var(--line-strong); background:var(--surface); color:var(--fg); cursor:pointer}
.sidenav > ol{display:none; position:absolute; left:16px; right:16px; top:100%; max-height:70vh; overflow:auto; padding:8px; background:var(--surface); border:1px solid var(--line-strong); border-radius:12px; box-shadow:0 8px 24px var(--shadow)}
.sidenav.open > ol{display:block}
main [id]{scroll-margin-top:64px}
/* wide: left pane, always open */
@media (min-width:1024px){
  .layout{display:grid; grid-template-columns:220px minmax(0, 1fr); gap:48px; max-width:1320px; margin:0 auto}
  .content{min-width:0; margin:0} /* no auto margins: they size the column to its widest table and the page scrolls sideways */
  .sidenav{top:32px; align-self:start; max-height:calc(100vh - 64px); overflow-y:auto; margin:0; padding:0; background:none; border:0}
  .sidenav-btn{display:none}
  .sidenav-title{display:block; margin:0 0 8px 14px; font-size:.75rem; letter-spacing:.08em; text-transform:uppercase; color:var(--muted)}
  .sidenav > ol{display:block; position:static; max-height:none; overflow:visible; padding:0; background:none; border:0; border-left:1px solid var(--line); border-radius:0; box-shadow:none}
  .sidenav a{margin-left:-1px}
  main [id]{scroll-margin-top:16px}
}
```

The negative top margin and `top:32px` assume `body` has 32px of top padding and 16px side padding; match them to the page's actual padding.

```js
var nav = document.getElementById('sidenav');
var navBtn = nav.querySelector('.sidenav-btn');
function setOpen(open) { nav.classList.toggle('open', open); navBtn.setAttribute('aria-expanded', String(open)); }
navBtn.addEventListener('click', function () { setOpen(!nav.classList.contains('open')); });
nav.addEventListener('click', function (e) { if (e.target.closest('a')) setOpen(false); });
document.addEventListener('click', function (e) { if (!nav.contains(e.target)) setOpen(false); });
document.addEventListener('keydown', function (e) { if (e.key === 'Escape' && nav.classList.contains('open')) { setOpen(false); navBtn.focus(); } });

var links = Array.prototype.slice.call(nav.querySelectorAll('a'));
var targets = links.map(function (a) { return document.getElementById(a.getAttribute('href').slice(1)); });
function setActive(i) {
  var a = links[i];
  links.forEach(function (l) { l.setAttribute('aria-current', String(l === a)); });
  nav.querySelectorAll('.in-path').forEach(function (li) { li.classList.remove('in-path'); });
  // mark the active link's <li> and its section's <li> (with collapse-subs, this opens that section's parts)
  for (var li = a.parentElement; li; li = li.parentElement.closest('li')) li.classList.add('in-path');
  // keep the active link in view when the pane itself scrolls (wide layout only)
  if (nav.scrollHeight > nav.clientHeight && (a.offsetTop < nav.scrollTop || a.offsetTop + a.offsetHeight > nav.scrollTop + nav.clientHeight)) {
    nav.scrollTop = a.offsetTop - nav.clientHeight / 2;
  }
}
// active = the last section or part whose top has passed 30% of the viewport; the last one at the very bottom
var current = -1, queued = false;
function update() {
  queued = false;
  var line = window.innerHeight * 0.3, i = 0;
  while (i + 1 < targets.length && targets[i + 1].getBoundingClientRect().top <= line) i++;
  if (window.innerHeight + window.scrollY >= document.documentElement.scrollHeight - 2) i = targets.length - 1;
  if (i !== current) { current = i; setActive(i); }
}
window.addEventListener('scroll', function () { if (!queued) { queued = true; requestAnimationFrame(update); } }, { passive: true });
window.addEventListener('resize', update);
update();
```

### Guides and fill-in fields

A guide is a doc the reader follows step by step: a how-to, a setup or install procedure, a runbook, a migration checklist. It uses this outline instead of the explainer one:

1. **Header** — title, one line on what the reader ends up with, and the remember/reset control if the guide has fields.
2. **Prerequisites** — what must already be installed, running or known, with versions.
3. **Steps** — one `<h3>` per step, numbered in the heading ("3. Start Valkey"), each with what to do, the exact command, and what success looks like. Steps are destinations, so they get nested links in the nav.
4. **Verify** — how to confirm the whole thing worked.
5. **Troubleshooting** — the likely failures, each with its symptom and its fix.
6. **References** — same rules as any doc.

A short background section before Prerequisites is fine when the reader needs context, but keep it brief; the steps are the point.

**Fill-in fields.** When a value the reader supplies (a version, host name, port, path, project name, ticket number) appears in two or more places, or a step has the reader gather details to paste somewhere else, make it a field instead of telling the reader to edit each command. Fields work in any doc, not just guides. Read [guide-fields.md](guide-fields.md) for the markup, CSS and JS before writing one, and follow these rules:

- **One field per value, at first use.** The input sits inline in the step where the value is first needed, with a label and a realistic example as its placeholder. Every later use is a highlighted token that shows the value (or `<name>` until it's filled in) and jumps back to the field when clicked.
- **Copy buttons on every code block** copy the filled-in text. If a block uses a value that's missing or fails its format check, the button sends the reader to that field instead of copying.
- **Format checks** where a value has a known shape (a version like 1.2.3, a port, a ticket key like OPS-123): a `pattern` plus a one-line hint, shown next to the field when the entry doesn't match.
- **Choice fields** when commands differ by platform or option (OS, shell, package manager): a `<select>` placed before the first step it affects, preset to the likeliest option for this reader, with each variant block marked `data-when`. A difference one sentence can cover doesn't need a choice field.
- **Output blocks** when the reader has to hand details to someone or something else (a ticket note, a config file, a message to ops): one block that assembles the values into paste-ready text, with its own Copy button.
- **Sharing the result.** The values bar also has **Save as PDF** (prints the guide with the reader's values and chosen variants filled in), **Download filled copy** (a new `.html` file with the values built in, for sending to a colleague) and **Copy values** (a plain-text `Label: value` list for a ticket or chat). Every doc with fields gets all three.
- **Remembering is the reader's choice.** Values are saved in the browser for that doc by default. A "Remember my values on this device" checkbox turns that off (and deletes what was saved), and a Reset button clears everything.
- **Never make a field for a secret.** Passwords, tokens, API keys and private keys must not be typed into the page, saved, or copied by it. Have the command read an environment variable the reader sets in their own shell (`$env:PGPASSWORD`, `"$PGPASSWORD"`), or leave a plain placeholder they replace after pasting, and say which.
- **No field for a value the author already knows.** Just write it.

### Verify before delivering

Before calling it done:
- Open the file in a real browser (the built-in browser pane, or Claude in Chrome) — don't just eyeball the source.
- Check both light and dark mode actually render correctly. With nothing saved, the page should match the system setting (emulate it with the pane's `colorScheme` option if available); the toggle should cycle System → Light → Dark, and a Light/Dark choice should survive a reload while System should not leave a saved key.
- Check the nav at a wide width (the left pane is visible without clicking anything and stays put while the page scrolls) and at a narrow width (it becomes the top bar, the "Sections" button opens and closes the list, and a jumped-to heading isn't hidden under the bar). In both, links work and the current-section highlight updates on scroll. If the nav lists parts, scroll through a multi-part section: the part highlight follows, the section link stays marked, and with `collapse-subs` only that section's parts are open.
- If the doc has fill-in fields: type a value and watch every use of it update; press Copy on a block with a value missing and with a malformed one (it should jump to the field, not copy); switch each choice field and check the right blocks show; reload with remembering on (values stay) and off (they're gone); press Reset. Then fill some values and check the share buttons: Copy values gives the `Label: value` list; Download filled copy saves a file that opens with those values and only one set of Copy buttons per block; print preview (Save as PDF) shows the values, only the chosen variants and the values table, with no inputs, nav or buttons.
- Check the browser console for errors.
- If the page has a CSS-grid diagram, check it at a narrow width too — grids that assume desktop width should degrade to a stacked layout rather than breaking.
- Confirm there are no external network calls (no CDN `<script src>`, no remote fonts/images) — everything must be inline.

## Markdown version

Only when Markdown was chosen. Mirror the HTML's content and heading structure in plain Markdown: same sections, same tables (as Markdown tables), same scenario walkthroughs, same references list. Drop anything that's purely visual/interactive (the theme toggle, the nav menu, CSS-grid diagrams) — render a diagram as a simple ASCII/text sketch or just describe the flow in prose instead. The goal is that someone reading only the `.md` file gets the same information, just without the visual polish.

For a doc with fill-in fields, put a **Values used in this guide** table right after Prerequisites (or at the top of a non-guide doc): the placeholder (`<valkey-version>`), what it is, an example, its format, and which steps use it. Commands and output blocks keep the `<name>` placeholders. Steps that depend on a choice field show every variant, each labeled (*Windows (PowerShell)*, *Linux or macOS (bash)*).

## PDF version

Only when PDF was chosen. Print it from the finished, verified HTML so the two never drift apart.

**Print stylesheet.** Every HTML doc gets an `@media print` block, whatever formats were chosen: it makes the reader's own Ctrl+P (and a guide's **Save as PDF** button) look right, and it's what the PDF step prints with:

```css
@media print{
  /* always print the light palette, whatever the screen theme: repeat the light vars here, or rely on the PDF step's light emulation */
  .sidenav, .theme-btn, .values-bar, .copy-btn{display:none !important}
  .layout{display:block; max-width:none}
  body{padding:0}
  pre{white-space:pre-wrap; overflow-wrap:anywhere}  /* long lines wrap instead of being cut off */
  h2, h3{break-after:avoid}
  pre, table, figure, .field{break-inside:avoid}
  a[href^="http"]::after{content:" (" attr(href) ")"; font-size:.8em; color:var(--muted)} /* links stay usable on paper */
}
```

A doc with fill-in fields also needs the print rules and print-only values table from [guide-fields.md](guide-fields.md). The PDF this step makes is printed blank (a fresh browser profile has no saved values), so its tokens show `<name>` placeholders and every choice variant appears under its label — the same content as the Markdown file. A reader who wants a PDF with their own values uses the guide's **Save as PDF** button.

**Producing the file.** Use a headless Chromium browser — Microsoft Edge is on every Windows machine; Chrome or Chromium elsewhere:

```bash
msedge --headless --disable-gpu --no-pdf-header-footer --user-data-dir="<scratch-dir>/edge-profile" --print-to-pdf="C:/path/topic-name.pdf" "file:///C:/path/topic-name.html"
```

Always pass a throwaway `--user-data-dir`: without it, if the reader already has Edge/Chrome open, the command hands off to that browser, exits 0, and writes no PDF. On Windows, run it through PowerShell's `Start-Process -Wait` so the call waits for the file. Check the PDF exists afterwards. (`chrome`/`google-chrome`/`chromium` take the same flags; on Windows the binaries are usually under `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` and `C:\Program Files\Google\Chrome\Application\chrome.exe`.) Headless printing uses the light `prefers-color-scheme` by default, so a System-themed page prints light. If no Chromium browser is available, say so and offer the HTML for the reader to print with Ctrl+P instead of reaching for another converter.

**Check it**: open the PDF (or render a page or two to an image) and confirm the nav and buttons are gone, nothing is cut off at the right edge, tables and code blocks aren't split awkwardly, and it's in the light palette.

## Claude artifact version

Only when Claude artifact was chosen. Publish the finished, verified HTML as it is; the snippets in this skill already handle the artifact frame. Read the artifact server's `guide` (topic `pages`) once per session before the first publish, and follow it over this section if they ever disagree.

**What changes inside an artifact**, and how the page copes:

- **Printing is refused.** A guide's **Save as PDF** button hides itself (the fields script detects the artifact by `window.claude`). Ctrl+P in the viewer prints claude.ai, not the doc, so if the user also wants a PDF, choose PDF as a format too.
- **Direct downloads are refused.** **Download filled copy** goes through the `downloads` capability instead, and the viewer confirms each save. Declare it for any doc with fill-in fields (below); without it the button hides itself.
- **Copy buttons work** (clipboard writes from a click are allowed; the code already catches a refusal).
- **Storage may be missing.** `localStorage` keeps the theme and remembered values for one viewer in one browser, and may throw; the snippets wrap every call.
- **Section ids**: links only pass a plain `#name`, and claude.ai reserves some names. Never use `settings`, `sidebar`, `customize`, `upgrade`, `downloads` or anything starting `comment-` as an `id`.
- **Nothing loads from outside** — same as the offline rule, so no change. Links to other sites stay plain `<a href>`; the viewer opens them outside the page.

**Publishing**:

1. Write the HTML file first — in the docs folder when HTML was also chosen, otherwise in the same place (the file is the artifact's source, and later updates republish it; tell the user where it is).
2. Publish it with `publish_files` (`index.html` ← that file), with the doc's `<title>` as `title` and a one-word `icon` such as `book`. Without a files tool, use `publish_version` with the whole HTML as `content`.
3. If the doc has fill-in fields, call `declare_capabilities` with `{"downloads": true}` on the new artifact, using the version the publish returned as `base_version`.
4. **Updating a doc that already has an artifact**: republish to the same artifact (`slug` + `base_version` from `get_artifact`, or find it with `list_artifacts` by title) rather than making a new one, so its link and sharing stay the same. If a capability is being added, declare it before publishing the page that uses it.
5. Give the user the link. New artifacts are private to them; they share it from the artifact's page on claude.ai (the tools can't change sharing).

**Check it**: open the link in the browser pane (the user may need to sign in to claude.ai there). Check the theme follows the system setting and the toggle works, the nav jumps and highlights, and for a guide: tokens update as you type, Copy values copies, Save as PDF is hidden, and Download filled copy brings up the save confirmation (decline it — no need to download).

## Where to save and how to deliver

- **Project documentation** (about one project: its architecture, setup, stack, a decision made for it): save the chosen files into that project's `docs/` folder, creating it if it doesn't exist.
- **Research documentation** (cross-project research, tool or vendor comparisons, build-vs-buy studies with no single project as its home): ask the user where it goes, or use a general documentation folder if one is already established in the conversation.
- Name every file the same way aside from extension (`topic-name.html`, `topic-name.md`, `topic-name.pdf`). Deliver only the formats that were chosen — a scratch HTML built just to print a PDF stays out of the docs folder.
- Deliver files directly when the session can present them (`present_files`, `SendUserFile` or similar), and say where they were saved. An artifact is delivered as its link.

## A note on tone

These are explainer docs, often read by someone who isn't already deep in the topic. Write like you're teaching, not logging. Favor plain language over jargon where you can, and when you do use a technical term, it's fine to define it briefly in passing rather than assuming it's known. Guides keep the same tone with shorter, imperative steps: say what to do, then what the reader should see.
