# Fill-in fields: starting point

Read this when a doc gets fill-in fields (see "Guides and fill-in fields" in [SKILL.md](SKILL.md) for when and why). It's a tested starting point: copy the pieces, adapt the colors to the page's own variables, keep the behavior.

## Markup

The values bar goes once, in the header: the remember/reset control plus three ways to share the filled-in result.

```html
<div class="values-bar">
  <label><input type="checkbox" id="values-remember" checked> Remember my values on this device</label>
  <button type="button" id="values-reset">Reset values</button>
  <button type="button" id="values-pdf">Save as PDF</button>
  <button type="button" id="values-download">Download filled copy</button>
  <button type="button" id="values-copy">Copy values</button>
</div>
```

- **Save as PDF** opens the browser's print dialog (choose "Save as PDF" as the printer). The print shows the reader's values in every command, only the variants they chose, and a table of their values; inputs, nav and buttons are hidden.
- **Download filled copy** saves a new `.html` file (`<name>-filled.html`) with the values built in as the fields' defaults. Whoever opens it sees the values, can still edit them, and Reset returns to them. It keeps its own storage, so it never mixes with the original's saved values.
- **Copy values** copies a plain-text list (`Label: value`, one per line, under the doc's title) for pasting into a ticket, chat or email. Each code and output block keeps its own Copy button for pasting commands.

The same page works as a published Claude artifact, where the frame refuses printing and direct downloads: the script hides **Save as PDF** there, and **Download filled copy** goes through the artifact's `downloads` capability (the viewer confirms the save). Declare that capability when publishing (see SKILL.md); if it's missing, the button hides itself.

A print-only table of the values goes right after the header, one row per field, with a token in each value cell (a choice field's token shows the chosen option's text, and its row has class `choice`):

```html
<table class="print-only values-table">
  <caption>Values used in this copy</caption>
  <tr><th scope="row">Valkey version</th><td><span class="fv" data-field="valkey-version"></span></td></tr>
  <tr class="choice"><th scope="row">Your system</th><td><span class="fv" data-field="os"></span></td></tr>
</table>
```

A text field goes inline, in the step where its value is first needed. `data-field` is the value's name; it's also what an empty token shows (`<valkey-version>`) and what the Markdown file uses as the placeholder. The hint is shown only when the entry doesn't match `pattern`.

```html
<div class="field">
  <label for="f-valkey-version">Valkey version</label>
  <input id="f-valkey-version" data-field="valkey-version" placeholder="8.1.1" pattern="\d+\.\d+\.\d+"
         autocomplete="off" spellcheck="false" aria-describedby="f-valkey-version-hint">
  <p class="field-hint" id="f-valkey-version-hint" hidden>Use the full version, like 8.1.1.</p>
</div>
```

A choice field sits before the first step it affects. Mark the likeliest option `selected`; Reset returns to it.

```html
<div class="field">
  <label for="f-os">Your system</label>
  <select id="f-os" data-field="os" autocomplete="off">
    <option value="windows" selected>Windows (PowerShell)</option>
    <option value="unix">Linux or macOS (bash)</option>
  </select>
</div>
```

Every later use of a value is a token, in prose or inside code:

```html
<pre><code>docker run -d --name valkey -p <span class="fv" data-field="valkey-port"></span>:6379 valkey/valkey:<span class="fv" data-field="valkey-version"></span></code></pre>
```

A block that only applies to some choices gets `data-when="field=value"` (several values: `data-when="os=unix,wsl"`) and a `data-label` naming the variant. Put it on a block element: a `<pre>`, `<p>`, `<div>` or `<li>`. On screen only the chosen variant shows; a blank print (nothing filled in, such as the PDF the skill generates) shows every variant under its label.

```html
<pre data-when="os=windows" data-label="Windows (PowerShell)"><code>...PowerShell version...</code></pre>
<pre data-when="os=unix" data-label="Linux or macOS (bash)"><code>...bash version...</code></pre>
```

An output block assembles values into paste-ready text:

```html
<figure class="output">
  <figcaption>Send to the team</figcaption>
  <pre><code>Host: <span class="fv" data-field="host"></span>
Valkey <span class="fv" data-field="valkey-version"></span> on port <span class="fv" data-field="valkey-port"></span></code></pre>
</figure>
```

Every `<pre>` inside `<main>` gets a Copy button from the script; don't add buttons by hand.

## CSS

Uses `--warn` and `--warn-soft` for missing or malformed values and `--accent-soft` for the field background; define them in both themes if the page doesn't have them.

```css
[hidden]{display:none !important}
.values-bar{display:flex; flex-wrap:wrap; align-items:center; gap:8px 16px; margin:14px 0 0; font-size:.85rem; color:var(--muted)}
.values-bar label{display:inline-flex; align-items:center; gap:6px; cursor:pointer}
.values-bar button{font:inherit; padding:3px 12px; border-radius:999px; border:1px solid var(--line-strong); background:var(--surface); color:var(--fg); cursor:pointer}
.field{display:flex; flex-wrap:wrap; align-items:center; gap:6px 12px; max-width:72ch; margin:12px 0 16px; padding:10px 14px; border-radius:10px; background:var(--accent-soft); transition:box-shadow .2s}
.field label{font-weight:650; min-width:16ch} /* lines the inputs up when labels differ in length */
.field input, .field select{font:inherit; font-family:var(--f-mono); font-size:.9rem; flex:1 1 16ch; min-width:0; max-width:36ch; padding:5px 10px; border:1px solid var(--line-strong); border-radius:6px; background:var(--surface); color:var(--fg)}
.field input[aria-invalid="true"]{border-color:var(--warn)}
.field-hint{flex-basis:100%; margin:0; font-size:.85rem; color:var(--warn)}
.field.attention{box-shadow:0 0 0 3px var(--accent)}
.fv{font-family:var(--f-mono); color:var(--accent); background:var(--accent-soft); border-radius:3px; padding:0 2px; cursor:pointer}
.fv.empty, .fv.invalid{color:var(--warn); background:var(--warn-soft)}
.fv.invalid{text-decoration:underline wavy}
.code{position:relative}
.code pre{padding-right:96px}
.copy-btn{position:absolute; top:8px; right:8px; max-width:calc(100% - 16px); font:inherit; font-size:.78rem; padding:3px 10px; border-radius:999px; border:1px solid var(--line-strong); background:var(--surface); color:var(--fg); cursor:pointer}
@media (max-width:640px){ .code pre{padding-right:16px; padding-top:44px} } /* phones: the button sits above the first line instead of covering it */
.output{margin:16px 0 20px}
.output figcaption{margin:0 0 6px; font-weight:650; color:var(--fg)}
.values-table caption{text-align:left; font-weight:650; margin:0 0 6px}
@media screen{ .print-only{display:none !important} }
@media print{
  .values-bar, .field, .copy-btn{display:none !important}
  .code pre{padding:12px 16px}
  .fv{background:none; padding:0; color:inherit}
  html:not(.filled) .values-table tr.choice{display:none} /* a blank print shows every variant, so no choice to report */
  .fv.empty{color:inherit; font-style:italic}
  /* blank print (nothing filled in): every variant, each under its label */
  html:not(.filled) [data-when]{display:block !important}
  html:not(.filled) li[data-when]{display:list-item !important}
  html:not(.filled) [data-when][data-label]::before{content:attr(data-label); display:block; margin:0 0 4px; font-weight:650; font-size:.85rem}
}
```

This goes alongside the page's general `@media print` block from SKILL.md (which hides the nav and theme button).

## JS

Values are saved in the browser's `localStorage`, never written into the HTML file: they stay on that browser on that device, and someone you send the file to sees empty fields (that's what **Download filled copy** is for). Set `NAME` to the doc's file name without extension: it's the storage key and the downloaded copy's name. Chrome and Edge give every `file://` page the same storage, so without a unique key two guides would share values.

**Make this the last `<script>` in `<body>`**, after the nav and theme-button code: its first line records the page's HTML for **Download filled copy**, and only what the browser has read so far gets recorded, so a script placed after it would be missing from the copy. (What the nav and theme scripts have already changed by then, the current-section mark and the button label, is recomputed when the copy opens.)

**Mark every one of the doc's own `<script>` tags `data-doc`** (`<script data-doc>`), including the theme script in `<head>`. The filled copy keeps only marked scripts, because a Claude artifact frame injects its own runtime scripts, with session data, into the page; an unmarked doc script would be dropped from the copy.

```js
(function () {
  var SOURCE = '<!DOCTYPE html>\n' + document.documentElement.outerHTML; // first line of the last script: every script is read, none of this one's changes made yet
  var NAME = 'valkey-postgres-guide';
  var filledStamp = document.documentElement.getAttribute('data-filled'); // set in downloaded filled copies
  var STORE = NAME + (filledStamp ? '-filled-' + filledStamp : '');      // each filled copy keeps its own storage
  var inArtifact = !!(window.claude && typeof window.claude.use === 'function'); // published as a Claude artifact
  var fields = {}, touched = {};
  document.querySelectorAll('input[data-field], select[data-field]').forEach(function (el) { fields[el.getAttribute('data-field')] = el; });
  var remember = document.getElementById('values-remember');
  var reset = document.getElementById('values-reset');

  function load(key) { try { return localStorage.getItem(STORE + '-' + key); } catch (e) { return null; } }
  function store(key, val) {
    try { if (val === null) localStorage.removeItem(STORE + '-' + key); else localStorage.setItem(STORE + '-' + key, val); } catch (e) {}
  }
  function value(name) { return fields[name] ? fields[name].value.trim() : ''; }
  function shown(name) { // what a token displays: a choice field shows its option's text
    var el = fields[name];
    return el && el.tagName === 'SELECT' ? el.options[el.selectedIndex].text : value(name);
  }
  function valid(name) { return !fields[name] || !fields[name].validity.patternMismatch; }
  function label(name) {
    var l = fields[name] && document.querySelector('label[for="' + fields[name].id + '"]');
    return l ? l.textContent.trim() : name;
  }

  // wrap every code block with a Copy button; data-when/data-label on the <pre> move to the wrapper so the button hides with it
  document.querySelectorAll('main pre').forEach(function (pre) {
    var wrap = document.createElement('div');
    wrap.className = 'code';
    ['data-when', 'data-label'].forEach(function (a) {
      if (pre.hasAttribute(a)) { wrap.setAttribute(a, pre.getAttribute(a)); pre.removeAttribute(a); }
    });
    pre.parentNode.insertBefore(wrap, pre);
    wrap.appendChild(pre);
    var btn = document.createElement('button');
    btn.type = 'button';
    btn.className = 'copy-btn';
    btn.textContent = 'Copy';
    wrap.appendChild(btn);
    btn.addEventListener('click', function () {
      var bad = Array.prototype.map.call(pre.querySelectorAll('.fv'), function (t) { return t.getAttribute('data-field'); })
        .filter(function (n) { return !value(n) || !valid(n); })[0];
      if (bad) { touched[bad] = true; render(); flash(btn, 'Fill in ' + label(bad) + ' first'); goTo(bad); return; }
      copyText(pre.textContent.replace(/\n$/, '')).then(function () { flash(btn, 'Copied'); }, function () { flash(btn, 'Copy failed: select the text'); });
    });
  });

  function render() {
    document.querySelectorAll('.fv').forEach(function (t) {
      var name = t.getAttribute('data-field'), v = value(name);
      t.textContent = v ? shown(name) : '<' + name + '>';
      t.classList.toggle('empty', !v);
      t.classList.toggle('invalid', !!v && !valid(name));
    });
    document.querySelectorAll('[data-when]').forEach(function (el) {
      var rule = el.getAttribute('data-when').split('=');
      el.hidden = rule[1].split(',').indexOf(value(rule[0])) === -1;
    });
    Object.keys(fields).forEach(function (name) {
      var bad = touched[name] && !!value(name) && !valid(name);
      var hint = document.getElementById(fields[name].id + '-hint');
      fields[name].setAttribute('aria-invalid', String(bad));
      if (hint) hint.hidden = !bad;
    });
  }
  function save() {
    if (!remember.checked) return;
    var vals = {};
    Object.keys(fields).forEach(function (name) { vals[name] = fields[name].value; });
    store('values', JSON.stringify(vals));
  }
  function goTo(name) {
    var el = fields[name];
    if (!el) return;
    var box = el.closest('.field') || el;
    el.scrollIntoView({ block: 'center' });
    el.focus({ preventScroll: true });
    box.classList.add('attention');
    setTimeout(function () { box.classList.remove('attention'); }, 1600);
  }
  function flash(btn, text) {
    if (!btn.restLabel) btn.restLabel = btn.textContent;
    btn.textContent = text;
    clearTimeout(btn.flashTimer);
    btn.flashTimer = setTimeout(function () { btn.textContent = btn.restLabel; }, 2200);
  }
  function valuesText() {
    return [document.title].concat(Object.keys(fields).map(function (name) {
      return label(name) + ': ' + (value(name) ? shown(name) : '(not filled in)');
    })).join('\n');
  }
  function downloadFilled() {
    // the untouched page, with the current values written in as the fields' defaults
    var doc = new DOMParser().parseFromString(SOURCE, 'text/html');
    Object.keys(fields).forEach(function (name) {
      var el = doc.getElementById(fields[name].id);
      if (el.tagName === 'SELECT') Array.prototype.forEach.call(el.options, function (o) { o.toggleAttribute('selected', o.value === fields[name].value); });
      else el.setAttribute('value', fields[name].value);
    });
    // keep only the doc's own scripts: an artifact frame injects its runtime (with session data) into <head>
    doc.querySelectorAll('script:not([data-doc])').forEach(function (s) { s.remove(); });
    doc.documentElement.removeAttribute('data-theme');
    doc.documentElement.setAttribute('data-filled', String(Date.now()));
    var html = '<!DOCTYPE html>\n' + doc.documentElement.outerHTML, file = NAME + '-filled.html';
    if (inArtifact) { // the artifact frame blocks <a download>; the downloads capability asks the viewer to confirm
      return window.claude.use('downloads').then(function (dl) {
        if (!dl) throw { code: 'unavailable' };
        return dl.save({ filename: file, data: html });
      });
    }
    var a = document.createElement('a');
    a.href = URL.createObjectURL(new Blob([html], { type: 'text/html' }));
    a.download = file;
    document.body.appendChild(a);
    a.click();
    a.remove();
    setTimeout(function () { URL.revokeObjectURL(a.href); }, 1000);
    return Promise.resolve();
  }
  function copyText(text) {
    // the clipboard API can be refused (a Claude artifact frame refuses it); fall back to the older copy command
    if (navigator.clipboard && window.isSecureContext) return navigator.clipboard.writeText(text).catch(function () { return legacyCopy(text); });
    return legacyCopy(text);
  }
  function legacyCopy(text) {
    return new Promise(function (ok, fail) {
      var ta = document.createElement('textarea');
      ta.value = text;
      ta.setAttribute('readonly', '');
      ta.style.position = 'fixed';
      ta.style.opacity = '0';
      document.body.appendChild(ta);
      ta.select();
      var done = false;
      try { done = document.execCommand('copy'); } catch (e) {}
      document.body.removeChild(ta);
      if (done) ok(); else fail();
    });
  }

  Object.keys(fields).forEach(function (name) {
    fields[name].addEventListener('input', function () { render(); save(); });
    fields[name].addEventListener('change', function () { touched[name] = true; render(); save(); });
  });
  document.addEventListener('click', function (e) {
    var t = e.target.closest('.fv');
    if (t) goTo(t.getAttribute('data-field'));
  });
  remember.addEventListener('change', function () {
    store('remember', remember.checked ? 'on' : 'off');
    if (remember.checked) save(); else store('values', null);
  });
  reset.addEventListener('click', function () {
    // back to the page's defaults: empty in the original, the built-in values in a filled copy
    Object.keys(fields).forEach(function (name) {
      var el = fields[name];
      if (el.tagName === 'SELECT') {
        var def = Array.prototype.filter.call(el.options, function (o) { return o.defaultSelected; })[0] || el.options[0];
        el.value = def.value;
      } else {
        el.value = el.defaultValue;
      }
    });
    touched = {};
    store('values', null);
    render();
  });

  var printFilled = false, pdfBtn = document.getElementById('values-pdf'), dlBtn = document.getElementById('values-download');
  pdfBtn.addEventListener('click', function () { printFilled = true; window.print(); });
  if (inArtifact) {
    pdfBtn.hidden = true; // the artifact frame ignores window.print
    window.claude.use('downloads').then(function (dl) { if (!dl) dlBtn.hidden = true; }); // capability not declared or not available here
  }
  // a print with values shows only the chosen variants; a blank one (Ctrl+P on an empty guide, headless PDF) shows them all
  window.addEventListener('beforeprint', function () {
    var any = Object.keys(fields).some(function (n) { return fields[n].tagName !== 'SELECT' && value(n); });
    document.documentElement.classList.toggle('filled', printFilled || any);
    printFilled = false;
  });
  dlBtn.addEventListener('click', function () {
    downloadFilled().then(function () { if (inArtifact) flash(dlBtn, 'Saved'); }, function (e) {
      if (e && e.code === 'declined') flash(dlBtn, 'Cancelled');
      else if (e && e.code === 'rate_limited') flash(dlBtn, 'Busy: try again');
      else { flash(dlBtn, 'Download not available here'); }
    });
  });
  var copyBtn = document.getElementById('values-copy');
  copyBtn.addEventListener('click', function () {
    copyText(valuesText()).then(function () { flash(copyBtn, 'Copied'); }, function () { flash(copyBtn, 'Copy failed'); });
  });

  Object.keys(fields).forEach(function (name) { if (value(name)) touched[name] = true; }); // built-in values in a filled copy

  remember.checked = load('remember') !== 'off';
  if (remember.checked) {
    var saved = {};
    try { saved = JSON.parse(load('values') || '{}'); } catch (e) {}
    Object.keys(saved).forEach(function (name) {
      var el = fields[name];
      if (!el || (el.tagName === 'SELECT' && !Array.prototype.some.call(el.options, function (o) { return o.value === saved[name]; }))) return;
      el.value = saved[name];
      if (saved[name]) touched[name] = true;
    });
  }
  render();
})();
```

## Notes

- **One input per value.** Two inputs with the same `data-field` would fight; later uses are always tokens.
- **Don't put a field inside a `data-when` block** unless the value is only used in that variant; otherwise clicking a token can't reach a hidden field.
- **`pattern` matches the whole entry** (the browser anchors it) and modern browsers compile it with the `v` flag, so escape `-` inside a character class (`[A-Z0-9\-]`). Check each pattern against its own placeholder example.
- **The filled copy never contains secrets**, because fields never hold them (see SKILL.md). It does contain everything else the reader typed, such as host names, as does the PDF.
- **`autocomplete="off"`** on every field stops the browser restoring values on reload by itself, which would defeat the remember switch.
- **The script needs `main` to wrap the doc body** (as the page layout already does) so the copy buttons stay off the nav and header.
