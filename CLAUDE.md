# CLAUDE.md — Anchor

> Guidance for Claude Code (and future-me) when working in this repo.
> Read this fully before touching `Anchor.html`. The constraints
> here are load-bearing — several of them are the *whole point* of the project,
> not incidental choices.

---

## 1. What this is

**Anchor** is a single-file, offline-first personal note hub for a
service-desk / customer-support workflow. It was built for Joey (Junior
Customer Support Engineer) as a personal productivity tool — a place to log
tickets, dump thoughts, paste screenshots, and keep canned responses, all
without friction.

The current starting point is a single HTML file — `Anchor.html` —
containing HTML + CSS + vanilla JavaScript inline, with no build step and no
dependencies. It's being grown into a proper multi-file web app (see §2), but
the offline, local-first, dependency-light spirit carries forward.

### Why it exists (the human context — keep this in mind)

The tool is designed around **ADHD / Autism-friendly principles**. This is the
soul of the project. Every feature and every UI decision should serve these:

- **Capture before it escapes.** Working memory is precious. The ⚡ brain-dump
  bar is always visible and saves on a single Enter. Never add steps between
  "I have a thought" and "it's saved."
- **Low cognitive load.** One clear "what needs me now" screen. Predictable,
  consistent layout. Generous whitespace. No clutter, no surprise motion.
- **Reduce decision fatigue.** Dropdowns over free text where sensible,
  snippets over re-typing, sensible defaults (new tickets default to
  Medium / New).
- **Single next action.** Each ticket has exactly one `nextAction` field —
  the next physical thing to do — surfaced on the Now screen. Avoid designs
  that force the user to re-plan from scratch.
- **Visible progress = dopamine.** Status counts, checklist progress bars,
  "Resolved ✓" tallies. Small wins should feel like wins.
- **Calm sensory design.** Muted palette, dark mode, `prefers-reduced-motion`
  honoured. No harsh contrast, no flashing, no aggressive reds except true
  urgency.

When in doubt about a feature, ask: *does this reduce friction and cognitive
load, or add to it?* If it adds, redesign or drop it.

---

## 2. Architecture & guardrails (read before refactoring)

This started life as a single self-contained HTML file — a deliberately scrappy
prototype optimised for "double-click and it runs." **That phase is ending.**
Joey is evolving it into a proper multi-file web app, so Claude Code is
*encouraged* to split the code into separate files, add modules, helper
scripts, build tooling, and real structure wherever it makes the code clearer,
more reusable, or more testable.

What changed: the old "stay one file / no modules / no build" rules are
**lifted.** What did **not** change is the *spirit* of the project. The
guardrails below are about values, not file count.

### Still in force (the values)
1. **Local-first & private by default.** The user's data lives on their device.
   Do **not** add analytics, telemetry, or third-party calls that leak data.
   You may add a backend or sync **only** as a deliberate, discussed feature
   (see the evolution path below) — never as a silent default. Offline must
   keep working.
2. **No dependency rot.** Prefer a small, well-chosen set of dependencies over
   a sprawling tree. Pin versions. Anything added should still build/run years
   from now. Vanilla-first; reach for a library when it earns its place.
3. **Accessibility & calm design are non-negotiable** (see §1). Dark mode,
   reduced-motion, keyboard support, and low cognitive load survive every
   refactor.
4. **Don't break existing user data.** Whatever the storage layer becomes,
   honour the schema-versioning + migration rule (§4) and the Backup/Restore
   contract.

### Now allowed / encouraged
- **Multiple files**: split HTML, CSS, and JS into separate files and folders.
- **ES modules** (`import`/`export`) and/or a bundler. (Note: ES modules don't
  load over `file://` — you'll need a local dev server, see §7.)
- **A build step** (Vite recommended — fast, minimal config, great DX) once the
  app outgrows no-build.
- **A framework** if it genuinely helps. The render-string + manual re-wire
  pattern (§5) is the first thing that will hurt as features grow; a small
  reactive layer (or Svelte/Vue/React) is a reasonable cure. Pick one and
  commit — don't half-migrate.
- **Helper scripts** (data migration, build, lint, test) — put them in
  `scripts/` or wire them into `package.json`.
- **TypeScript** is welcome once there's a build step; it'll pay off as the data
  model grows.

### Suggested evolution path (incremental, low-risk)
1. **Extract, no build.** Pull the CSS into `styles/`, the JS into ES modules
   under `src/` (`state.js`, `render/`, `editor.js`, …), load via
   `<script type="module">`, serve with a static dev server. Stays fully
   debuggable with zero tooling.
2. **Introduce Vite (+ optional TypeScript)** once modules stabilise — adds
   HMR, bundling, type safety, and a path to a deployable build.
3. **Componentise** the views (Now / Tickets / Notes / Snippets) and the ticket
   editor, retiring the manual re-wire pattern.
4. **(Optional) Going server-side.** If multi-device sync or accounts become a
   goal, design the API and auth explicitly with Joey, keep an offline mode,
   and treat user-data privacy as a first-class requirement. This is a product
   decision, not a drive-by change.

Keep a working build at every step — land changes incrementally so the app is
never broken for long.

---

## 3. File map

The whole app is `Anchor.html` (~52 KB). Top-to-bottom:

```
<head>
  <style> … </style>          ← all CSS. Uses CSS custom properties for theming.
</head>
<body>
  <div class="app"> … </div>   ← sidebar nav + main (topbar, quickbar, #content)
  <div class="overlay">        ← ticket editor modal (populated by JS)
  <div class="lightbox">       ← full-screen image preview
  <div class="toast">          ← transient confirmation messages
  <script> (IIFE) </script>    ← ALL logic lives here, at end of body
</body>
```

### Target multi-file structure (as it grows — see §2)

Once extracted, a clean layout to aim for (adapt as the code dictates):

```
anchor/
├─ index.html              ← shell: mounts the app, loads the entry module
├─ styles/
│  ├─ tokens.css           ← CSS custom properties (theme vars, colours)
│  └─ app.css              ← layout + component styles
├─ src/
│  ├─ main.js              ← boot: load state, apply theme, first render
│  ├─ state.js             ← load()/save()/seed(), schema + migrations
│  ├─ constants.js         ← STATUSES, PRIORITIES, CATEGORIES
│  ├─ util.js              ← uid, esc, now, fmtDate, toast, …
│  ├─ render/              ← one module per view (now, tickets, notes, snippets)
│  ├─ editor.js            ← ticket editor modal
│  ├─ images.js            ← paste / drop / downscale handling
│  └─ backup.js            ← export / import
├─ scripts/                ← build / migration / dev helper scripts
└─ package.json            ← once a build step (Vite) is introduced
```

Let the split follow the natural seams in the code (§4 data, §5 render/wire,
the editor, images, backup). Don't force structure ahead of need.

### CSS conventions
- **Theming via CSS variables** on `:root` and `html[data-theme="dark"]`.
  Never hardcode a colour in a rule — add/use a `--var`.
- **Logical properties** (`margin-inline-start`, `inset-inline-end`,
  `text-align: start`) are used throughout so RTL/future-proofing is free.
  Keep using them; avoid `left`/`right`.
- Status colours: `--new`, `--prog`, `--waitc`, `--waitv`, `--done` (+ `-bg`
  variants). Priority colours: `--urgent`, `--high`, `--med`, `--low`.
- `@media (prefers-reduced-motion: reduce)` kills all transitions — respect it.

---

## 4. Data model

Single `state` object, persisted as JSON in `localStorage[anchor_v1]` (renamed
from the legacy `jm_support_hub_v1`; `load()` migrates the old key automatically
via the `LEGACY_KEYS` list).

```js
state = {
  settings: { theme: "light" | "dark" },

  tickets: [{
    id,            // uid() — stable unique id
    no,            // human ticket ref, e.g. "T-001" (editable)
    title,
    customer,      // contact name
    phone,         // rendered as a tel: click-to-call link
    email,         // rendered as a mailto: link
    status,        // one of STATUSES[].k
    priority,      // one of PRIORITIES  ("Urgent"|"High"|"Medium"|"Low")
    category,      // one of CATEGORIES (or "")
    desc,          // description / what's happening
    steps,         // running work log
    nextAction,    // the ONE next thing — surfaced on Now screen
    followUp,      // timestamp (ms) | null — callback/follow-up time
    links: [{ label, url }],
    images: [ dataURL ],   // base64 JPEG/PNG, downscaled (see §6)
    tags: [ string ],
    checklist: [{ text, done }],
    pinned: bool,
    created, updated       // timestamps (ms)
  }],

  notes: [{ id, text, created, pinned, done }],   // brain-dump items

  snippets: [{ id, title, cat, body }]            // canned responses
}
```

Constants defined at top of the IIFE: `STATUSES` (5), `PRIORITIES` (4),
`CATEGORIES` (10). `statusMeta(k)` maps a status string → `{k, cls, dot}`.

### Schema versioning
The storage key ends in `_v1`. **If you make a backwards-incompatible change
to the shape, bump to `_v2` and write a migration** in `load()` that reads the
old key, transforms, and saves under the new one. Never silently break
existing users' data — `load()` already does exactly this for the
`jm_support_hub_v1` → `anchor_v1` rename via the `LEGACY_KEYS` list; follow
that same pattern. `seed()` provides the first-run starter content.

---

## 5. Code patterns & conventions (follow these)

The JS is organised as: **helpers → state load/save → render functions →
event wiring → boot**. Key patterns:

- **`save()` after every mutation.** Any change to `state` must be followed by
  `save()`. It wraps `localStorage.setItem` in try/catch and toasts on quota
  failure. Don't mutate-and-forget.

- **Render = build an HTML string, set `innerHTML`, then re-wire.** Views are
  pure functions returning HTML strings (`viewNow()`, `viewTickets()`, …).
  `render()` picks the active view, sets `#content.innerHTML`, then calls
  `wireView()`.
  ⚠️ **THE #1 GOTCHA:** setting `innerHTML` destroys all event listeners.
  Any interactive element rendered this way must be (re)bound in `wireView()`
  (or `wireEditorDynamic()` inside the modal). There is **no event
  delegation** — it's explicit re-binding. If you add a button in a view's
  HTML, add its listener in the matching wire function or it will do nothing.
  ➜ This manual re-wire is the pattern most likely to creak as the app grows;
  it's the prime candidate to retire when you componentise or adopt a small
  reactive layer / framework (§2). Until then, keep the wire-after-render
  discipline intact.

- **Always escape user content with `esc()`** before putting it in an HTML
  string. Tickets, notes, snippets — all user text. This prevents broken
  layout and is a basic XSS hygiene habit even for a local tool.

- **The ticket editor edits a deep-cloned `draft`, not the live object.**
  `openTicket()` clones via `JSON.parse(JSON.stringify(...))`. Changes only
  hit `state` on **Save** (`saveTicket()`). Closing the modal discards. This
  "edit a copy, commit on save" pattern is intentional — preserve it so
  Cancel/Esc always safely discards.

- **`uid()`** generates ids. **`now()`** is the timestamp helper. **`toast(msg)`**
  shows a 2.2s confirmation — use it for every user action so the app always
  acknowledges input (important for the ADHD use case: never leave the user
  wondering if something registered).

- **Keyboard shortcuts** live in one `keydown` handler: `/` search, `Q`
  brain-dump, `N` new ticket, `1–4` switch views, `Esc` close. They no-op
  while typing in a field. Keep new shortcuts in that handler and update the
  in-app help if you add one.

- **`prompt()` / `confirm()` are used** for the snippet editor and delete
  confirmations. Fine for a local file. (If this ever moves to a hosted/
  sandboxed context, replace them with custom modals — but see §2.3.)

---

## 6. Important gotchas

- **`localStorage` size limit (~5 MB per origin).** Images are by far the
  biggest consumer. `addImageFile()` → `downscale()` caps width at **1400px**
  and re-encodes to **JPEG quality 0.85** to keep storage lean. Keep this.
  If users hit the cap, `save()` toasts a warning. A future improvement could
  move images to IndexedDB (much larger quota) — see backlog.
- **file:// origin quirk.** In Chromium, each file *path* is its own storage
  origin. If the user moves/renames the HTML file, their data appears to
  "vanish" (it's tied to the old path). **This is exactly why Backup/Restore
  exists** (`#btn-export` / `#btn-import`, JSON download/upload). Mention this
  to users; never treat localStorage as the only copy.
- **No build = test by opening the file.** There's no test suite. Verify
  changes by opening the file in a browser and exercising the flow. If you
  want a parse sanity-check without a browser: extract the final `<script>`
  body and run it through `new Function(body)` in Node — it throws on syntax
  errors. (That's a *parse* check only, not a behaviour check.)
- **Backup/restore replaces all state** after a confirm. It validates that the
  imported JSON has `tickets` and `notes` before accepting.

---

## 7. How to run / develop

```
# There is no build. To "run":
open Anchor.html                  # or just double-click it

# Quick JS syntax sanity check (Node, optional):
node -e "const fs=require('fs');const s=fs.readFileSync('Anchor.html','utf8');\
const m=s.match(/<script>([\s\S]*)<\/script>\s*<\/body>/);new Function(m[1]);console.log('OK');"
```

**Running a multi-file version (once you split into ES modules):** browsers
block `import` over `file://`, so serve the folder over HTTP during dev:

```
python3 -m http.server 5173      # then open http://localhost:5173
npx serve .                      # zero-config static server
# or VS Code "Live Server", or `npm run dev` once Vite is set up
```

**Editing workflow (single-file phase):** the file is large but flat. Use search to jump to a
function (e.g. `function viewTickets`, `function saveTicket`). Make the change,
re-open in browser, exercise the affected flow, confirm `localStorage` still
loads (test with existing data — don't only test on a fresh seed).

**Before committing a change, self-check:**
- [ ] No external references introduced (still single-file, offline).
- [ ] `save()` called after any new state mutation.
- [ ] New rendered interactive elements are wired in `wireView()` /
      `wireEditorDynamic()`.
- [ ] User text passes through `esc()`.
- [ ] Schema change? → version bump + migration in `load()`.
- [ ] Still respects reduced-motion and dark mode (use CSS vars).
- [ ] Tested with *existing* saved data, not just a fresh state.

---

## 8. Feature backlog / ideas (not yet built)

Loosely ordered by value-to-effort. Confirm with Joey before big ones.

**Quick wins**
- Per-ticket "time logged" / quick timer for call duration.
- Duplicate-a-ticket action (for recurring issue types).
- Inline snippet editor modal (replace the `prompt()` chain).
- Drag-to-reorder checklist steps.
- "Copy ticket summary" button (formats a ticket as text for pasting into a
  real ticketing system / email).
- Soft-delete with an Undo toast instead of hard `confirm()` delete.

**Medium**
- **Agenda / calendar view** of follow-ups (day/week) — high value for the
  callback workflow.
- Saved filters / saved searches.
- Tag manager + filter-by-tag chips.
- Markdown rendering in description / notes.
- Keyboard-only ticket creation flow.

**Larger**
- **Move images to IndexedDB** to escape the ~5 MB localStorage cap (keep
  localStorage for the lightweight text state; reference images by id).
- Auto-backup reminder (e.g. nudge to export weekly) — stays offline.
- Optional encrypted export.
- "Templates" for common ticket types (pre-filled checklist + category).
- PWA wrapper so it installs as a desktop app (still fully offline) — only if
  it doesn't compromise the single-file simplicity.

**Guardrails (the goals have expanded — multi-file & build are now welcome)**
- Frameworks, bundlers, ES modules, TypeScript: all fair game now (see §2).
- A backend / accounts / sync: allowed, but only as a deliberate, discussed
  feature — design it with Joey, keep an offline mode, and treat user-data
  privacy as a hard requirement. Never a silent default.
- Still avoid: analytics / telemetry, third-party calls that leak user data,
  and over-engineering ahead of need. Add structure because the code needs it,
  not for its own sake.

---

## 9. Tone for changes

This is a *passion project* and a personal tool. Bias toward:
- Keeping it delightful, calm, and frictionless over feature-complete.
- Small, well-contained changes that are easy to review and land incrementally.
- Preserving the ADHD/Autism design principles in §1 above all else.

When proposing a change, lead with how it reduces friction or cognitive load
for the day-to-day support workflow — that's the bar every feature has to clear.
