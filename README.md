# Unfold

A phone-first reading app that delivers books one idea at a time, in three layers of depth. Built as plain HTML/JS with zero dependencies, zero build step, and zero backend. Progress lives in the reader's browser (localStorage).

**If you are an AI that has been asked to add a book to this project: this README is your complete instruction set. Read all of it before writing anything.**

**Prime directive — digestible, never abridged.** This app compresses *form* (layers, one idea at a time), never *content*. A book is done only when its full essential content is represented: every distinct idea the author would defend as part of the book. If you find yourself "selecting highlights" or "picking the best concepts," stop — you are building an abridgement, which is a failure state. **This mistake has been made twice in this project's history** and had to be repaired both times: once when books shipped at ~50–70% coverage, and again when whole chapters of three books had no concept carrying them. Both times it looked like reasonable editorial judgment from the inside. It is not. Write the coverage map first (Step 2) — it is the only thing that reliably catches this.

---

## How the app works

- `index.html` is the entire app. Do not modify it to add books.
- `manifest.json` is the PWA manifest (home-screen install). Not to be confused with `books/manifest.js`.
- `books/manifest.js` lists the book files to load. **Its order is the library's order.**
- `books/*.js` — one file per book. Each file calls `window.registerBook({...})` with the book's data.
- `sw.js` caches everything for offline reading. It has a `CACHE_VERSION` constant.
- `icon.png` (180px) / `icon-512.png` — the block-meter mark. Regenerate both if the identity changes.
- Reader progress (read/saved/position) is stored in localStorage under `unfold-state-v1`. Book files never touch it.

Current library: Naval (24), Meditations (22), Principles (20), Psycho-Cybernetics (23) — 89 concepts.

A book is a set of **concepts**. Each concept has three layers the reader unfolds one tap at a time:

1. **hook** — creates pull. Shown immediately.
2. **idea** — the complete thought, standalone.
3. **deep** — mechanism, evidence, application.

Books have one of two **modes**:

- `"shuffle"` — for books whose chapters don't depend on order (essay collections, aphorisms, principle lists). Reader gets a random unread concept; the action slab shuffles to the next.
- `"sequential"` — for books whose ideas build on each other. Reader moves with prev/next; position is remembered.

---

## Adding a book: the procedure

### Step 0 — Source material

Acceptable sources, in order of preference:

1. A PDF/EPUB the user has uploaded (they own it — you may read it fully to distill it).
2. Your own reliable knowledge of the book, if it's well-known and you know it deeply.
3. Public-domain full texts (e.g., pre-1929 works, classic philosophy in old translations).

**Distillation policy (non-negotiable).** All `hook`, `idea`, and `deep` text must be **original distillation written by you** — your own words, structure, and framing. Do not reproduce the book's prose. Verbatim quotes: for in-copyright works, keep them under 15 words and at most one per book file; public-domain works may be quoted a little more freely.

This holds even though the app is private and personal-use. Two reasons, and the second is the real one: reproducing prose at length is not something to do regardless of audience, and — more importantly — **an excerpt viewer is a worse product.** The whole value here is that someone did the work of compressing a 300-page argument into a thing you can hold in one thought. Pasting the author's paragraphs back in is the failure this app exists to solve. Distillation *is* the product.

### Step 1 — Classify the mode

Ask: *if the reader landed on chapter 14 first, would it make sense?*

- Yes → `"shuffle"` (Almanack of Naval, Meditations, Atomic Habits, most essay/wisdom books)
- No, ideas build on prior ideas → `"sequential"` (Principles, most narratives, staged arguments)

When genuinely mixed, prefer `"shuffle"` and use `section` fields to give structure.

### Step 2 — Map the book completely, then extract every idea

This step is where the process once failed: an early build covered only ~50–70% of each book by selecting highlights. Never again. The procedure:

1. **Outline the full structure** — every part and chapter of the source.
2. **Extract every distinct idea.** Books repeat each idea many times through different stories; repetition collapses into one concept, but a distinct idea is never dropped. A typical 300-page book yields 15–35 concepts. The book decides the count — there is no target number, and a low count is a red flag, not a virtue.
3. **Write a coverage map:** every chapter/section → the concept(s) that carry it. A chapter may share a concept with other chapters (that's compression), but no chapter may map to nothing (that's abridgement). Pure-anecdote chapters that only illustrate an existing idea are marked "absorbed by <concept-id>."
4. Only when the map has **zero orphans** do you start writing.

Write the coverage map as a comment block at the top of the book file (see any existing book for the format). It is part of the deliverable, not scratch work — it's how the next person verifies you didn't abridge.

Give each concept a `section` (the book's part/theme) so the Contents view has structure. Keep each section's concepts **contiguous** in the array — Contents groups by section, so scattered sections still render correctly, but a contiguous array is what makes the reading order sane in sequential books.

### Step 3 — Write the three layers

This is the craft. The voice contract:

**hook** (15–30 words)
- One or two sentences. Creates tension, curiosity, or a reframe. Never summarizes the resolution.
- Good: "Two people work equally hard. One earns a hundred times more. The difference isn't talent or luck."
- Bad: "This chapter explains the concept of leverage and its three types." (a label, not a hook)

**idea** (80–130 words, 1–2 paragraphs separated by blank lines)
- The complete thought, fully standalone. A reader who stops here should genuinely have the idea — not a teaser for the deep layer.
- Plain declarative prose. No bullet points, no headers, no "in this chapter."

**deep** (180–280 words, 2–4 paragraphs)
- What the idea layer earns you: the mechanism (*why* it works), the strongest example or evidence from the book, honest limits/failure modes, and one concrete application the reader can act on — typically as the closing paragraph, often opening with "Application:".
- Do not restate the idea layer. Deepen it.

**source** (short string) — where in the book this lives, e.g. `"Part I · Building Wealth"` or `"Book 4"`.

**Voice throughout:** direct, plain verbs, zero filler, no listicle cadence, no "the author argues that…" scaffolding on every line. Second person is welcome in applications. Write like a sharp friend who read the book carefully, not like a book report. Density over length — every sentence must carry information.

### Step 4 — Create the file

Create `books/<id>.js` where `<id>` is a short lowercase slug (e.g. `atomic-habits.js`):

```js
window.registerBook({
  id: "atomic-habits",            // unique slug, matches filename
  title: "Atomic Habits",
  author: "James Clear",
  mode: "shuffle",                // or "sequential"
  concepts: [
    {
      id: "ah-01",                // "<short-prefix>-<number>", unique within book
      section: "Fundamentals",    // groups concepts in the Contents view
      title: "…",                 // the idea's name, not the chapter's name
      hook: "…",
      idea: "First paragraph.\n\nSecond paragraph.",   // \n\n = paragraph break
      deep: "…\n\n…\n\nApplication: …",
      source: "Chapter 1"
    }
    // …
  ]
});
```

Escape rules: the file is plain JS — escape internal double quotes (`\"`) or use typographic quotes ("…"), and use `\n\n` for paragraph breaks. No trailing commas issues to worry about in modern browsers, but keep it clean.

### Step 5 — Register it

Add the filename to the array in `books/manifest.js`.

### Step 6 — Bump the cache

In `sw.js`, increment `CACHE_VERSION` (e.g. `v6` → `v7`) and add the new book file to the `ASSETS` list. If you skip this, offline readers won't get the new book.

### Step 7 — QA

Run the contract check — it catches every mechanical failure in seconds. Save as `check.js` anywhere and run `node check.js`:

```js
const fs = require("fs"), path = require("path");
const dir = "<path-to>/unfold/books";
const books = [];
global.window = { registerBook: b => books.push(b), BOOK_FILES: null };
eval(fs.readFileSync(path.join(dir, "manifest.js"), "utf8"));
for (const f of window.BOOK_FILES) eval(fs.readFileSync(path.join(dir, f), "utf8"));
const wc = t => (t || "").trim().split(/\s+/).filter(Boolean).length;
for (const b of books) {
  const ids = new Set();
  for (const c of b.concepts) {
    const p = [];
    if (ids.has(c.id)) p.push("DUPLICATE ID"); ids.add(c.id);
    if (!c.section || !c.source || !c.title || !c.hook || !c.idea || !c.deep) p.push("MISSING FIELD");
    const [h, i, d] = [wc(c.hook), wc(c.idea), wc(c.deep)];
    if (h < 15 || h > 30) p.push(`hook ${h}w`);
    if (i < 80 || i > 130) p.push(`idea ${i}w`);
    if (d < 180 || d > 280) p.push(`deep ${d}w`);
    if (p.length) console.log(`${b.id} ${c.id}: ${p.join("; ")}`);
  }
  console.log(`${b.title}: ${b.concepts.length} concepts`);
}
```

Then the judgment calls the script can't make:

- [ ] Script reports zero issues (word budgets, unique ids, no missing fields)
- [ ] Mode classification justified (shuffle vs sequential)
- [ ] **Coverage map has zero orphans** — every chapter/section maps to a concept or is explicitly absorbed by one. Total coverage, not highlights.
- [ ] Every hook creates pull without spoiling; every idea stands alone; every deep adds mechanism + application
- [ ] No reproduced prose; quote budget respected
- [ ] Filename added to `books/manifest.js`; book file added to `ASSETS` and `CACHE_VERSION` bumped in `sw.js`

### Extending an existing book

Books can grow. Append new concepts to the book's `concepts` array following the same contract, keep `id`s unique and sequential, and bump `CACHE_VERSION`. Never rewrite existing concepts a reader may have already read/saved unless asked — their progress is keyed to concept `id`s.

---

## Deployment

Hosted on GitHub Pages. Any push to the default branch redeploys automatically. On a phone, open the Pages URL and use Share → **Add to Home Screen** — the app then runs standalone and works offline via the service worker.

## Design notes (for anyone touching index.html)

**Direction: industrial minimalism — bone, core, oxide.** Sportswear-catalogue rather than book-app: enormous tight-tracked uppercase grotesque, monospace utility labels, hairline rules, hard edges. Nothing is rounded anywhere — `border-radius: 0` is enforced in the reset, and that is a design decision, not an oversight. No shadows, no gradients, no ornament.

**Palette** (light / dark, defined once as CSS custom properties):

| token | light | dark | used for |
|---|---|---|---|
| `--paper` | `#DCD7CB` bone | `#0E0D0B` core | page |
| `--raised` | `#EDEAE2` salt | `#1A1917` | panels |
| `--ink` | `#0E0D0B` | `#DCD7CB` | text, filled blocks, the slab |
| `--mid` | `#6E695C` concrete | `#857F71` | mono labels |
| `--line` | `#BFB8A7` | `#2C2A24` | hairlines, empty blocks |
| `--oxide` | `#8C3F1D` | `#C2542A` | **saved things only** — never decorative |

Dark mode is a true inversion, not a dimming. Both schemes clear WCAG AA.

**Type is two families doing three jobs.** One grotesque (`Helvetica Neue`/system) for everything readable — set at weight 700, `letter-spacing: -0.035em`, uppercase, `line-height: 0.96` for titles; regular weight at 1.055rem/1.72 for body. One monospace for every label, count, and piece of metadata — 0.645rem, `0.16em` tracking, uppercase. If a string is data *about* the reading rather than the reading itself, it is monospace. No exceptions; that split is the whole system.

**The signature is the block meter.** One block per idea in the book: hollow = unread, solid ink = read, solid oxide = saved, oxide ring = where you are. It appears on every library row (glanceable progress) and above every concept (tappable — each block jumps to that idea). It is simultaneously the progress bar, the position indicator, and a navigation control, which is why it earns the space. **Do not add a second progress indicator anywhere.**

**The page is the button.** There is nothing pinned to the bottom of the screen — tapping anywhere on the concept advances it: hook → idea → deeper → the next idea. A reader finishes a whole book by tapping the same place repeatedly. The tap handler ignores taps that land on a control, taps that follow a scroll or swipe (>10px of movement), and taps made while text is selected. A quiet `TAP TO CONTINUE` line teaches the gesture and permanently retires itself after six taps (`S.flags.taps`). The last concept of a sequential book shows `END OF BOOK` and refuses to advance further.

**Controls live at the top, in two rows.** Row one: `← LIBRARY`, then `READ` / save icon / `CONTENTS`. Row two: prev arrow, block meter, next arrow — the arrows flank the meter because that's what they move through, and prev is hidden entirely in shuffle books. `READ` is a real toggle: reading auto-marks it, and tapping un-marks, which also clears the book's completion flag so finishing again still celebrates. The save icon is a bookmark that fills oxide.

**Section tag.** Each concept opens with its section in a small outlined pill — no fill, and the only rounded thing in the app, deliberately. No position counter in the body; the block meter carries that.

**Motion is mechanical.** Snap easing (`cubic-bezier(0.2,0,0,1)`), short durations, no bounce, no pulsing, nothing breathes. Layers open by animating `grid-template-rows: 0fr → 1fr`. Concept changes cross-fade in 120ms. Finishing a book inverts the screen to a full-bleed COMPLETE card. `prefers-reduced-motion` kills all of it.

**Watch CSS specificity when adding controls.** `.reader-top button.mono` styles the plain text buttons; `.readbtn` / `.savebtn` size themselves. A bare `.reader-top button` rule will out-specify the class rules and silently crush their padding — this has already happened once.

**Structure carries meaning.** Numbering appears only where order is real information (sequential position, contents index). Library rows are labelled by mode and idea count, never by a decorative catalogue number.

Contents and Saved are full-screen panels sliding from the bottom. The phone back button walks the stack (panel → reader → library) via `history.pushState`. Swipe left/right moves between ideas; arrow keys do the same on desktop.
