# generate-docs

A skill that generates **project documentation** and **research
documentation**, such as architecture overviews, stack and tool comparisons,
decision docs and research write-ups, plus step-by-step **guides** (how-tos,
setup steps, runbooks). You pick the formats — HTML, Markdown, PDF, a Claude
artifact, or any mix; if you don't say, Claude asks, defaulting to HTML +
Markdown. The files share one name:

- **`topic.html`**: self-contained, with inline CSS and JS and no network
  calls, so it opens offline by double-clicking. It follows your system's
  light/dark setting, with a toggle that cycles System, Light and Dark, and
  a section menu that highlights the section you're reading: a left pane on
  wide screens, a drop-down from a top bar on narrow ones. Sections with
  several parts get nested links to each part.

  Guides can have **fill-in fields** for values only you know, such as a
  version, port or host name. You type each value once, in the step that
  first needs it, and every command and paste-ready block that uses it updates.
  Every code block has a Copy button, which sends you to the right field if a
  value is missing or in the wrong format. A choice field (Windows or Linux,
  say) shows only the matching commands. Values are remembered in your
  browser's local storage (not in the file, so they don't travel with it)
  unless you turn that off, and Reset clears them. Passwords and keys are
  never fields.

  To share what you filled in, a guide has three buttons: **Save as PDF**
  (your values and chosen commands, ready to print or attach), **Download
  filled copy** (a new HTML file with your values built in, for a
  colleague to open and keep editing) and **Copy values** (a plain
  `Label: value` list to paste into a ticket or chat).
- **`topic.md`**: the same content in plain Markdown, for git, a wiki or a
  chat paste.
- **`topic.pdf`**: a print of the HTML, without the menu and buttons, for
  attaching or printing. Made with headless Edge or Chrome.
- **Claude artifact**: the same HTML published as a private claude.ai page,
  so you can share a link instead of a file (share it from the artifact's
  page on claude.ai). Theme, menu, fields and copy buttons all work there;
  Save as PDF is hidden, because artifacts can't print, and Download filled
  copy asks you to confirm the save. Needs the claude.ai artifacts connector
  in the session.

The skill tells Claude to:

- gather and check the real facts first, for example by reading a tool's
  LICENSE file before calling it free;
- use real tables, CSS-grid diagrams and "what happens when…" walkthroughs;
- always end with a **References** section, including official links and any
  known lookalike domains;
- open the result in a browser before handing it over.

The full instructions are in [`SKILL.md`](SKILL.md).

## When it runs

Claude picks it up when you ask to document, write up or explain something,
or to create project documentation, research documentation, or an overview,
comparison or decision doc for a technical topic. "Write this up",
"document this" or "create an overview of X" is enough; you don't need to name
a format. It's aimed at architecture explainers, build-vs-buy comparisons and
cross-project research, not quick answers in chat.

To call it directly, type `/generate-docs` and choose it from the list, then
add the topic. When it's installed as a plugin, its full name is
`/generate-docs:generate-docs`.

## Where it saves

- **Project documentation** (about one project) goes in that project's
  `docs/` folder, which is created if needed.
- **Research documentation** (cross-project research, tool comparisons,
  build-vs-buy studies) goes in a documentation folder you name. If you
  haven't named one, Claude asks.

## Where it works

| Where | Works? | How to get it there |
| --- | --- | --- |
| Claude Code CLI, desktop Code tab, VS Code | ✔ full skill | Install from the marketplace on this computer (path 1), or sync it from your claude.ai account |
| claude.ai web, desktop Chat, Cowork, mobile | ✔ HTML and Markdown, but the browser check and saving to `docs/` are skipped; the files come back as downloads. PDF only if the sandbox has a Chromium browser | On your claude.ai account (path 2): add the `primelogiclabio/claude-tools` marketplace under **Customize › Plugins**, upload the folder as a zip, or keep it as an account skill |
| Claude Code cloud sessions (claude.ai/code) | ✔ only if it's on your claude.ai account | Path 2 |

Outside Claude Code, skills need a Pro, Max, Team or Enterprise plan, with
**Code execution and file creation** turned on in **Settings › Capabilities**.

The **Claude artifact** format is offered only in sessions that have the
claude.ai artifacts connector (its `publish_files` / `publish_version`
tools). Without it, the format question leaves that choice out.

## Install

One line installs it, in either of two places:

- **Inside a Claude Code session** (CLI or desktop Code tab, 2.1.275+), type:
  ```
  /plugin install generate-docs --marketplace primelogiclabio/claude-tools
  ```
- **From a terminal** (PowerShell, bash; CLI 2.1.292+), run:
  ```
  claude plugin install generate-docs --marketplace primelogiclabio/claude-tools
  ```

Both add the `claude-tools` marketplace first if you haven't added it yet.

The step-by-step instructions for both paths are in the
[repo README](https://github.com/primelogiclabio/claude-tools#installing). Notes specific to this skill:

- **As a plain skill, without the plugin wrapper.** Link the folder into your
  skills directory. Put your clone's path in place of `C:\src\claude-generate-docs`:

  ```powershell
  New-Item -ItemType Junction -Path "$HOME\.claude\skills\generate-docs" -Target "C:\src\claude-generate-docs"
  ```

  Remove the link only with `cmd /c rmdir "$HOME\.claude\skills\generate-docs"`.
  Never use `Remove-Item -Recurse` on a junction.
- **Avoid loading it twice.** If this skill is already saved to your claude.ai
  account under **Customize › Skills**, it already reaches Chat, Cowork and
  mobile, and it syncs into signed-in Claude Code sessions as `generate-docs`.
  Installing the plugin as well adds a second copy, `generate-docs:generate-docs`,
  and the two then compete to answer the same requests. Keep one.
- **Renamed from `reference-doc`.** Earlier copies used that name. Remove any
  old one so it doesn't compete with this one:
  - an old `~/.claude/skills/reference-doc` junction: remove it with
    `cmd /c rmdir`;
  - an old `reference-doc@claude-tools` install: run
    `claude plugin uninstall reference-doc@claude-tools`;
  - an old `reference-doc` account skill: turn it off.
- **It needs a browser.** It has no runtime dependencies, but its check step
  expects one: the desktop app's built-in browser pane, or Claude in Chrome.
  Checking an artifact also needs you signed in to claude.ai in that browser.
- **PDF needs Edge or Chrome** on the computer, which Claude runs headless to
  print the HTML. Edge comes with Windows. Without either, Claude says so and
  gives you the HTML to print yourself.

## Developing

From this folder:

```sh
claude plugin validate .
```

It's a skill with no code, so there are no tests. To try a change, ask for a
write-up in a new session. To try the HTML snippets in an artifact, publish a
test page built from them, open it signed in to claude.ai, then delete it.

When a change reaches users, bump `version` in `.claude-plugin/plugin.json`:
the marketplace moves its pin to a new commit only when the version changes.
If the `description` changes, update the marketplace entry in `claude-tools`
to match.

## Files

```
generate-docs/
├── .claude-plugin/plugin.json   lets this folder install as a plugin
├── SKILL.md                     the skill: frontmatter + instructions
├── guide-fields.md              starting code for fill-in fields, read only when a doc has them
└── README.md                    this file
```
