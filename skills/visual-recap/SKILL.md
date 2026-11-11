---
name: visual-recap
description: >-
  Turn a PR branch, commit range, or local git diff into ONE self-contained
  HTML file that gives a code reviewer a visual overview of the change:
  TL;DR, before/after UI mockups, schema and API changes, a file map, and
  annotated key diffs. Local git only — no GitHub, no hosting, no MDX. Use
  when the user asks to "recap this PR/branch/diff", "make a visual recap",
  or wants help understanding a large change before reviewing it.
---

# Visual Recap

Summarize a diff **after** the work exists, at a higher altitude than
line-by-line review. The output is a single `.html` file the reviewer opens in
a browser: it shows the shape of the change — what moved, which contracts
changed, how the UI differs, where the risky lines are — so they can read the
raw diff afterwards with a map in their head.

## Output contract

The deliverable is exactly **one self-contained `.html` file**:

- All CSS in a single `<style>` block in `<head>`. No Tailwind, no CSS
  frameworks, no external stylesheets.
- Minimal vanilla JS — the small tab switcher in the template below is usually
  all of it. No JS libraries or frameworks.
- Offline-readable: no external images, fonts, or iframes. Inline SVG is fine.
- One allowed exception: for a very diff-heavy recap where syntax highlighting
  genuinely helps, you may load highlight.js (or diff2html) from a public CDN
  (jsdelivr/cdnjs) as progressive enhancement — the page must still be fully
  readable with no network. Default to **no CDN**.
- Write it as `recap-<branch>.html` in the repo root, or a path the user gives.
  It is a throwaway review aid — never commit it. After writing, open it
  (`open <file>` on macOS, `xdg-open` on Linux) and report the path.

## When to use / skip

Build a recap when the change is large, multi-file, UI-heavy, or touches
schema, API contracts, permissions, or architecture. Skip it for small,
single-file, or obvious diffs — a recap is review overhead, and a tiny change
reviews faster as a plain diff. Say so instead of producing a thin recap.

## Scope — local git only

Everything comes from the local repository. Do not call GitHub APIs, `gh`, or
any hosted service. If the user names a PR number with no local branch, ask
for a branch name or ref range instead.

Default scope is the whole branch versus its base:

```sh
# develop is this repo's base branch — use main/master where appropriate.
# The three-dot form diffs against the merge-base, so each command is
# self-contained (no shell variables that go stale between commands).
git log --oneline develop..HEAD              # the commits being recapped
git diff --stat develop...HEAD               # footprint: files + line counts
git diff -M develop...HEAD                   # the diff itself (-M detects renames)
git diff --name-status -M develop...HEAD     # A/M/D/R per file, for the file map
```

Sanity-check the footprint first: if `--stat` comes back empty, the range is
wrong (wrong base branch, detached state) — stop and fix the refs rather than
recapping an empty diff.

The user can narrow or change scope ("just the last commit", "include
uncommitted work", explicit `ref1..ref2`). Include uncommitted changes only
when asked or when they are clearly part of the same work unit. Read the
changed files themselves (not just hunks) whenever you need surrounding
context to describe a change accurately.

## Grounding rule

Structured content is trustworthy only if it is derived mechanically from the
real diff: real file paths, real field names, real method/path strings, real
before/after lines. Never invent, round, or guess — a confidently wrong recap
is worse than none, because a reviewer who trusts it may skip the exact line
it got wrong. Prose (the TL;DR, notes, risk callouts) is the only place you
write freely; mark anything inferred rather than extracted as inferred.

Never transcribe secrets. Diffs can contain API keys, tokens, webhook URLs, or
`.env` values — redact them (`sk-•••`, `<redacted>`) in every diff, mockup,
and note.

## Recap structure

One skeleton, top to bottom. Include a section only when the diff has content
for it; never pad with boilerplate ("this recap is an aid...", file counts in
prose — the header chips already carry that).

1. **Header** — title (≤ 70 chars), `branch → base`, commit count, files
   changed, +/− lines, date.
2. **TL;DR** — 1–3 short paragraphs: what changed, why, and what a reviewer
   should scrutinize (breaking changes, risky hunks, decisions visible in the
   diff).
3. **UI changes** — before/after mockups, when the diff changes rendered UI.
4. **API changes** — method + path + params/response tables per changed
   endpoint; mark added/removed/changed fields, show old types struck through.
5. **Data model changes** — one table per changed entity/table with per-field
   added/removed/modified flags, built from the actual migration/schema diff.
6. **Architecture** — only for real structural shifts: a small boxes-and-arrows
   diagram in plain HTML/CSS (flex/grid + borders) or inline SVG.
7. **File map** — every changed file with its A/M/D/R badge, a short note
   saying what changed in it, and its +/− counts.
8. **Key changes** — 3–8 tabs, one per load-bearing file: a one-line summary,
   the important hunks rendered as a unified diff, and a few annotation
   callouts on the lines that matter.

Budgets that keep it reviewable: 3–8 key-change tabs; ≤ ~150 diff lines per
tab (excerpt the hunks that matter, note what was skipped); a handful of
annotations per file, not one per line.

**Good:** a 25-file auth change → before/after mockups of the login surface, a
two-paragraph TL;DR, a sessions-table schema card, an endpoint card for the
new refresh route, a full file map, five annotated key-change tabs.
**Bad:** one giant unsegmented diff dump; or a three-block recap of a 40-file
change that forces the reviewer straight back into the raw diff.

## UI changes: recreate, don't copy

Show UI deltas as simple generic mockups rebuilt in basic HTML using the
`.mock` classes in the template. Do **not** render the repo's actual
components, import its CSS, or imitate its visual design — the mockup's job is
placement and content, and generic styling makes clear it's a sketch.

- Use **real text from the diff**: actual labels, button copy, column names,
  option values, error strings. No lorem ipsum, no gray bars.
- Show a **Before / After pair** when comparison clarifies the change; keep
  unchanged controls present in both frames so the reviewer can see exactly
  what moved. Use a single "After" frame for purely additive UI.
- For a flow, show the meaningful states in order: entry surface → opened
  interaction (dialog/popover/tab) → resulting state (including empty/error/
  permission-denied states the diff implements).
- Mock only the changed surface (a form section, a dialog, a table row) — not
  the whole page, unless placement in the page is itself the change.
- Label frames with `.mock-label` headings **above** the frame, never inside
  it.

## Diffs

Hand-render unified diffs with the template's `.diff` classes — no diff
library needed. Take the lines straight from `git diff`, keep the `+`/`-`/
space prefixes, and:

- **HTML-escape every code line** (`&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`)
  — one unescaped `<` in JSX or generics silently breaks the page.
- Keep hunk headers (`@@ … @@`) as `.ln.hunk` rows so the reviewer can find
  the spot in the real file.
- Interleave `.note` callouts between hunks for the annotations — what the
  hunk does and why it matters, not a restatement of the code.
- Long lines scroll inside the `<pre>` (`overflow-x: auto`); the page itself
  must never scroll horizontally.
- For a brand-new file, show the key excerpt as add-lines with annotations
  rather than dumping the whole file.

## HTML template

Start from this skeleton. The `<style>` block is the design system — extend it
if a section needs something new, but keep the token/class approach (colors
only via `var(--…)` so dark mode keeps working).

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Recap: {branch} — {short title}</title>
<style>
  :root {
    --bg:#ffffff; --fg:#1b1f24; --muted:#667085; --line:#e4e7ec;
    --card:#f8fafc; --accent:#4f46e5; --accent-soft:#eef2ff;
    --add-bg:#e6ffec; --add-fg:#1a7f37; --del-bg:#ffebe9; --del-fg:#cf222e;
    --hunk-bg:#f1f5f9;
    --mono:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg:#0d1117; --fg:#e6edf3; --muted:#8b949e; --line:#30363d;
      --card:#161b22; --accent:#8b93ff; --accent-soft:#20263f;
      --add-bg:#12261c; --add-fg:#3fb950; --del-bg:#2c1215; --del-fg:#f85149;
      --hunk-bg:#161b22;
    }
  }
  * { box-sizing:border-box }
  body { margin:0 auto; max-width:1080px; padding:40px 24px 80px;
    background:var(--bg); color:var(--fg);
    font:15px/1.6 system-ui,-apple-system,"Segoe UI",sans-serif }
  h1 { font-size:26px; margin:0 0 6px }
  h2 { font-size:19px; margin:44px 0 14px; padding-top:20px;
    border-top:1px solid var(--line) }
  h3 { font-size:15px; margin:22px 0 8px }
  p { margin:8px 0 }
  code { font-family:var(--mono); font-size:.92em }
  .meta { color:var(--muted); font-size:13px }
  .chips { display:flex; gap:8px; flex-wrap:wrap; margin:12px 0 }
  .chip { border:1px solid var(--line); border-radius:999px; padding:2px 10px;
    font-size:12.5px; background:var(--card) }
  .chip.add { color:var(--add-fg) }  .chip.del { color:var(--del-fg) }
  .card { border:1px solid var(--line); border-radius:8px;
    background:var(--card); padding:14px 16px; margin:12px 0 }

  /* tables (API params, schema fields) */
  table { border-collapse:collapse; width:100%; font-size:13.5px; margin:10px 0 }
  th, td { text-align:left; padding:6px 10px; border:1px solid var(--line) }
  th { background:var(--card); font-size:12.5px }
  .was { color:var(--del-fg); text-decoration:line-through }
  .method { font:700 11px var(--mono); border-radius:4px; padding:2px 6px;
    background:var(--accent-soft); color:var(--accent) }

  /* file map */
  .files { border:1px solid var(--line); border-radius:8px; overflow:hidden;
    margin:12px 0 }
  .file { display:flex; align-items:center; gap:10px; padding:8px 12px;
    border-top:1px solid var(--line); font-size:13.5px }
  .file:first-child { border-top:0 }
  .file .path { font-family:var(--mono); font-size:12.5px; white-space:nowrap }
  .file .desc { color:var(--muted); flex:1; min-width:0; overflow:hidden;
    text-overflow:ellipsis; white-space:nowrap }
  .badge { width:20px; height:20px; border-radius:4px; flex:none;
    display:inline-flex; align-items:center; justify-content:center;
    font:700 11px var(--mono) }
  .badge.A { background:var(--add-bg); color:var(--add-fg) }
  .badge.M { background:var(--accent-soft); color:var(--accent) }
  .badge.D { background:var(--del-bg); color:var(--del-fg) }
  .badge.R { background:var(--hunk-bg); color:var(--muted) }
  .delta { font-family:var(--mono); font-size:12px; white-space:nowrap }
  .delta .plus { color:var(--add-fg) }  .delta .minus { color:var(--del-fg) }

  /* UI mockups — deliberately generic, never the app's real styling */
  .mocks { display:grid; grid-template-columns:1fr 1fr; gap:20px; margin:16px 0 }
  .mocks.stack { grid-template-columns:1fr }   /* wide/desktop frames */
  .mocks > div { display:flex; flex-direction:column }
  .mock-label { font-size:12px; font-weight:600; text-transform:uppercase;
    letter-spacing:.05em; color:var(--muted); margin:0 0 6px }
  .mock { border:1.5px solid var(--line); border-radius:10px;
    background:var(--bg); padding:16px; display:flex; flex-direction:column;
    gap:10px; flex:1 }
  .m-row { display:flex; align-items:center; gap:8px; flex-wrap:wrap }
  .m-spacer { flex:1 }
  .m-btn { border:1.5px solid var(--line); border-radius:6px; padding:4px 12px;
    font-size:13px; background:var(--card) }
  .m-btn.primary { background:var(--accent); border-color:var(--accent);
    color:#fff }
  .m-input { border:1.5px solid var(--line); border-radius:6px; padding:5px 10px;
    font-size:13px; color:var(--muted); background:var(--bg) }
  .m-pill { border:1px solid var(--line); border-radius:999px; padding:1px 9px;
    font-size:11.5px }
  .m-pill.on { background:var(--accent-soft); border-color:var(--accent);
    color:var(--accent) }
  .m-muted { color:var(--muted); font-size:12.5px }
  .m-new { outline:2px solid var(--accent); outline-offset:2px;
    border-radius:6px }   /* ring around the added/changed control */

  /* tabs + diffs */
  .tab-bar { display:flex; gap:4px; flex-wrap:wrap; margin:16px 0 0;
    border-bottom:1px solid var(--line) }
  .tab { border:1px solid transparent; border-bottom:0;
    border-radius:6px 6px 0 0; background:none; color:var(--muted);
    padding:6px 12px; font:12.5px var(--mono); cursor:pointer }
  .tab.active { border-color:var(--line); background:var(--card);
    color:var(--fg) }
  .tab-panel { display:none }
  .tab-panel.active { display:block }
  .diff-summary { margin:14px 0 8px; font-size:14px }
  pre.diff { border:1px solid var(--line); border-radius:8px;
    background:var(--card); margin:8px 0; padding:8px 0; overflow-x:auto;
    font:12.5px/1.55 var(--mono); white-space:normal }
  .ln { display:block; padding:0 12px; white-space:pre }
  .ln.add { background:var(--add-bg) }
  .ln.del { background:var(--del-bg) }
  .ln.hunk { background:var(--hunk-bg); color:var(--muted); padding:2px 12px }
  .note { border-left:3px solid var(--accent); background:var(--accent-soft);
    border-radius:0 6px 6px 0; padding:8px 12px; margin:10px 0;
    font-size:13.5px }
</style>
</head>
<body>

<header>
  <h1>{Short title of the change}</h1>
  <p class="meta"><code>{branch}</code> → <code>{base}</code> · {N} commits · {date}</p>
  <div class="chips">
    <span class="chip">{N} files changed</span>
    <span class="chip add">+{added}</span>
    <span class="chip del">−{removed}</span>
  </div>
</header>

<h2>TL;DR</h2>
<p>{What changed and why; what to scrutinize.}</p>

<h2>UI changes</h2>
<h3>{Surface name}</h3>
<div class="mocks">
  <div>
    <p class="mock-label">Before</p>
    <div class="mock">
      <div class="m-row"><strong>Settings</strong><span class="m-spacer"></span><span class="m-btn">Save</span></div>
      <span class="m-input">example.com</span>
    </div>
  </div>
  <div>
    <p class="mock-label">After</p>
    <div class="mock">
      <div class="m-row"><strong>Settings</strong><span class="m-spacer"></span><span class="m-btn primary">Save</span></div>
      <span class="m-input">example.com</span>
      <div class="m-row m-new"><span class="m-pill on">301</span><span class="m-pill">302</span><span class="m-muted">Redirect type</span></div>
    </div>
  </div>
</div>

<h2>API changes</h2>
<div class="card">
  <p><span class="method">POST</span> <code>/api/example/route</code> — {one-line purpose}</p>
  <table>
    <tr><th>Field</th><th>Type</th><th>Change</th></tr>
    <tr><td><code>redirectType</code></td><td><code>301 | 302</code></td><td>added</td></tr>
    <tr><td><code>target</code></td><td><span class="was">string</span> <code>URL</code></td><td>type changed</td></tr>
  </table>
</div>

<h2>File map</h2>
<div class="files">
  <div class="file"><span class="badge A">A</span><span class="path">app/settings/Redirects.tsx</span><span class="desc">new settings tab</span><span class="delta"><span class="plus">+214</span></span></div>
  <div class="file"><span class="badge M">M</span><span class="path">proxy.ts</span><span class="desc">route new endpoint through auth</span><span class="delta"><span class="plus">+6</span> <span class="minus">−1</span></span></div>
</div>

<h2>Key changes</h2>
<div class="tabs">
  <div class="tab-bar">
    <button class="tab active">app/settings/Redirects.tsx</button>
    <button class="tab">proxy.ts</button>
  </div>
  <div class="tab-panel active">
    <p class="diff-summary">{One line: what this file's change does and why.}</p>
    <pre class="diff"><code>
<span class="ln hunk">@@ -12,6 +12,14 @@ export function Redirects() {</span>
<span class="ln">  const form = useForm&lt;RedirectSchema&gt;();</span>
<span class="ln del">-  const type = '301';</span>
<span class="ln add">+  const type = form.watch('redirectType');</span>
</code></pre>
    <div class="note">{Annotation: why this hunk is load-bearing.}</div>
  </div>
  <div class="tab-panel">
    <p class="diff-summary">…</p>
  </div>
</div>

<script>
for (const bar of document.querySelectorAll('.tab-bar')) {
  const tabs = [...bar.querySelectorAll('.tab')];
  const panels = [...bar.parentElement.querySelectorAll(':scope > .tab-panel')];
  bar.addEventListener('click', (e) => {
    const i = tabs.indexOf(e.target.closest('.tab'));
    if (i < 0) return;
    tabs.forEach((t, j) => t.classList.toggle('active', j === i));
    panels.forEach((p, j) => p.classList.toggle('active', j === i));
  });
}
</script>
</body>
</html>
```

Template notes:

- `pre.diff` uses `white-space:normal` with `white-space:pre` on each `.ln`
  span on purpose — it lets you put one `.ln` per source line without the
  newlines between spans rendering as extra blank lines. Keep each diff line's
  original `+`/`-`/space prefix and indentation inside the span.
- `.mocks` shows narrow frames side by side; add `.stack` for wide desktop
  surfaces so they aren't crushed into half-width columns.
- Every `.tab-panel` count must match its `.tab` count, in order — the tab
  switcher pairs them by index.

## Final checklist

Before reporting the file path:

1. Every code/diff line is HTML-escaped (`grep -n '<[a-zA-Z]' ` inside diff
   content you authored is a quick smoke test for stray unescaped tags).
2. Tab buttons and panels match 1:1 per `.tabs` group.
3. File map covers **every** changed file; key-change tabs cover the 3–8 that
   matter; anything intentionally skipped is noted.
4. All facts (paths, fields, endpoints, labels) came from the diff; secrets
   redacted; inferred statements marked as inferred.
5. No external requests (unless the explicit CDN exception was used, and the
   page still reads fine offline).
6. Open the file in the browser and report its absolute path.
