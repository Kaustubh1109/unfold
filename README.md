# Unfold

A phone-first reading app that delivers books one idea at a time, in three layers of depth. Built as plain HTML/JS with zero dependencies, zero build step, and zero backend. Progress lives in the reader's browser (localStorage).

**If you are an AI that has been asked to add a book to this project: this README is your complete instruction set. Read all of it before writing anything.**

**Prime directive — digestible, never abridged.** This app compresses *form* (layers, one idea at a time), never *content*. A book is done only when its full essential content is represented: every distinct idea the author would defend as part of the book. If you find yourself "selecting highlights" or "picking the best concepts," stop — you are building an abridgement, which is a failure state. This exact mistake was made once in this project's history and had to be repaired. Do not repeat it.

---

## How the app works

- `index.html` is the entire app. Do not modify it to add books.
- `books/manifest.js` lists the book files to load.
- `books/*.js` — one file per book. Each file calls `window.registerBook({...})` with the book's data.
- `sw.js` caches everything for offline reading. It has a `CACHE_VERSION` constant.
- Reader progress (read/saved/position) is stored in localStorage under `unfold-state-v1`. Book files never touch it.

A book is a set of **concepts**. Each concept has three layers the reader unfolds one tap at a time:

1. **hook** — creates pull. Shown immediately.
2. **idea** — the complete thought, standalone.
3. **deep** — mechanism, evidence, application.

Books have one of two **modes**:

- `"shuffle"` — for books whose chapters don't depend on order (essay collections, aphorisms, principle lists). Reader gets a random unread concept; an ensō button shuffles to the next.
- `"sequential"` — for books whose ideas build on each other. Reader moves with prev/next; position is remembered.

---

## Adding a book: the procedure

### Step 0 — Source material

Acceptable sources, in order of preference:

1. A PDF/EPUB the user has uploaded (they own it — you may read it fully to distill it).
2. Your own reliable knowledge of the book, if it's well-known and you know it deeply.
3. Public-domain full texts (e.g., pre-1929 works, classic philosophy in old translations).

**Copyright policy (non-negotiable):** All `hook`, `idea`, and `deep` text must be **original distillation written by you** — your own words, structure, and framing. Never reproduce the book's prose. Verbatim quotes: none for copyrighted works unless under 15 words, and at most one such quote in the entire book file. Public-domain works may be quoted more freely, but distillation is still the product — this app is not an excerpt viewer.

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

Give each concept a `section` (the book's part/theme) so the Contents view has structure. Order concepts sensibly even in shuffle books (Contents shows them in order).

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

In `sw.js`, increment `CACHE_VERSION` (e.g. `v3` → `v4`) and add the new book file to the `ASSETS` list. If you skip this, offline users won't get the new book.

### Step 7 — QA checklist

- [ ] File loads without console errors (valid JS, quotes escaped)
- [ ] `id` is unique; concept `id`s are unique
- [ ] Mode classification justified (shuffle vs sequential)
- [ ] **Coverage map has zero orphans** — every chapter/section of the source maps to a concept or is explicitly absorbed by one. Total coverage, not highlights.
- [ ] Every hook creates pull without spoiling; every idea stands alone; every deep adds mechanism + application
- [ ] No verbatim copyrighted prose; quote budget respected
- [ ] Word counts roughly within contract (hooks aren't essays, deeps aren't summaries)
- [ ] Filename added to `manifest.js`; `CACHE_VERSION` bumped in `sw.js`

### Extending an existing book

Books can grow. Append new concepts to the book's `concepts` array following the same contract, keep `id`s unique and sequential, and bump `CACHE_VERSION`. Never rewrite existing concepts a reader may have already read/saved unless asked — their progress is keyed to concept `id`s.

---

## Deployment

Hosted on GitHub Pages. Any push to the default branch redeploys automatically. On a phone, open the Pages URL and use Share → **Add to Home Screen** — the app then runs standalone and works offline via the service worker.

## Design notes (for anyone touching index.html)

Aesthetic: minimal zen / paper. Ink on rice paper, iOS-native serif (New York/Charter), hairline rules, one accent — the seal red, used only for saving. The signature element is the hand-drawn ensō circle used as the shuffle control. Keep everything else quiet: no shadows, no gradients, no decoration that doesn't encode meaning. Respect `prefers-reduced-motion`.
