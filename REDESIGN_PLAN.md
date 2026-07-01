# Portfolio Website Redesign Plan

Goal: evolve the current rough site into the themed, illustrated multi-page
experience sketched in `Webpage Theme Ideas.pdf`. Each page is its own themed
"room" (record player, tarot reading, plant nook, gallery wall, Mizzou Columns)
sharing one consistent visual language.

---

## 0. Recommended foundation: Astro

**Recommendation: rebuild on [Astro](https://astro.build).**

Why Astro fits this project specifically:

- **Component reuse without a heavy framework.** The nav menu is currently
  copy-pasted into all 5 HTML files. In Astro it becomes one `<Nav />`
  component used everywhere — edit once.
- **It's still just HTML/CSS.** Astro components are HTML + scoped CSS with a
  little templating. You keep learning the fundamentals; you're not forced into
  React-style thinking. Scoped `<style>` blocks per component prevent the
  "everything in one giant stylesheet" problem.
- **Ships zero JavaScript by default.** Pages load fast. You add interactivity
  only where needed (magic 8-ball, tarot flip, scroll animation) via "islands."
- **First-class GitHub Pages deploy.** Official adapter + Action; this also
  permanently fixes the case-sensitivity deploy bug (see §1).
- **Content collections** make the Publications page trivial later — list
  entries as data/Markdown, Astro renders them.

Alternatives considered: plain vanilla (simplest, but you'd hand-roll nav
includes and a build step yourself); a JS framework like React/Next (overkill —
this is a mostly-static content site, not an app). Astro is the sweet spot.

If at any point Astro feels like too much, everything below still applies to a
vanilla setup — only the "how shared components work" details change.

---

## 1. Bugs to fix immediately (independent of redesign)

These explain the `0a8c6f7 "troubleshooting liveview vs github pages discrepancy"`
commit:

1. **Image case mismatches** — break on GitHub Pages (case-sensitive Linux),
   work on Mac (case-insensitive):
   - `index.html` references `img/arm.PNG`; the file is `arm.png`.
   - `css/styles.css` references `../img/table.PNG`; the file is `table.png`.
   - Fix: make every reference match the real filename exactly. Going forward,
     standardize on all-lowercase asset filenames.
2. **Broken page skeletons** — `projects.html`, `friends.html`, `blog.html` are
   missing `<body>` tags (and all content). Astro's layout system replaces these
   anyway, so this resolves during the rebuild.
3. **Empty `js/java.js`** — currently unused; will be replaced by Astro islands.

---

## 2. Design system (do this first — it unifies the themes)

**Each page has its own, distinct color palette** — the record room, the tarot
table, the gallery wall, and the plant nook are meant to look and feel different.
What makes them read as *one site* is NOT shared colors; it's the shared
**typography, spacing scale, component structure, and overall craft level**. So
the design system locks the *unifying* layer while deliberately leaving color
per-page:

- **Per-page color palettes.** Define a separate palette for each page (pulled
  from your Pinterest / 80s-mixtape references and from each scene's mood —
  warm wood for home, dark/candlelit jewel tones for tarot, etc.). Implement as
  a theme attribute on `<body>` (e.g. `data-theme="tarot"`) that swaps the full
  set of color custom properties. Your existing `:root` colors become the Home
  palette.
- **Typography (shared).** Pick two fonts used site-wide: a retro/display face
  for headings (70s–80s character) and a clean, readable body face. Load via
  Google Fonts or self-host.
- **Spacing & sizing scale (shared).** A simple scale (e.g. 4/8/16/24/48px) keeps
  layouts consistent across pages.
- **Shared components** (Astro): `Layout.astro` (page shell, fonts, global CSS),
  `Nav.astro` (the menu), and small reusable bits as they emerge. Color comes
  from the active page theme, so the same components look at home in any room.

---

## 3. Page-by-page build plan

Order chosen by value + by "lock the visual language on the most-finished page
first."

### Phase A — Home (record player)  *(sets the bar)*
- **Scrap the hand-made `record.PNG` / `arm.png` / `table.png` images.** Instead,
  **draw the record player in pure CSS/SVG** — vinyl = concentric circles,
  tonearm = rounded rects, cabinet + knobs = simple shapes. Benefits: crisp at
  any size, free, fully themeable from the Home palette, and easy to animate
  (spin, tonearm drop, knob turn, teal label glow). Fallback if preferred: a
  free CC0 turntable photo/illustration (Unsplash/Openverse). Either way the
  layout slot is asset-agnostic, so swapping in custom art later is easy.
- Add the **text knobs** detail and the glowing teal record label.
- Build the **"about me" record-shelf** section: rotating/cycling photos of you
  + short bio + résumé link, styled like a record shelf.
- 80s/mixtape flourish: link to your Spotify playlists.
- *(Defer the scroll animation to Phase E.)*

### Phase B — Portfolio (tarot reading on cloth)
- Cloth/table background; **projects rendered as tarot cards** laid out like a
  spread.
- Candles with a soft glow; deck-styled nav placement.
- Empty/placeholder state: the playful "the cards aren't speaking to me, try
  again later" copy from your sketch.
- *(Defer card-flip interaction + magic 8-ball to Phase F.)*
- Decide: pull projects from a small data file so adding one = adding a card.

### Phase C — Art (gallery wall)
- Wall background with **framed art** in a salon-style hang.
- **Nav inside a wall clock** motif.
- Baseboard + wood floor at the bottom.
- Nicely cropped images of your art (consistent aspect ratios / mats).

### Phase D — Friends (plant nook)
- Each friend-feature as a **plant**: Spotify song recs and "leave a message for
  Emma."
- The message form needs a backend decision — see §4.

### Phase E — Delight layer (interactivity, added after layouts exist)
- Home: scroll animation (record-player top-down view → profile).
- Portfolio: tarot card flip reveal; **magic 8-ball** for the empty state.
- Friends: message submission.
- Each is a small Astro island (vanilla JS or a tiny component), kept isolated.
- Constraint: prefer the **simplest thing that runs for free** — plain vanilla
  JS / CSS animations over libraries or anything needing a paid host.

### Deferred — Publications (the Mizzou Columns)
*Nav slot is live (knob sits below Portfolio) but the page is a placeholder; the
full build is kept here for when you return to it.*
- Keep the **Mizzou's iconic Columns** styling — the campus landmark's columns
  serve as the structural backdrop, with publications displayed against /
  between them.
- Primary content: a list of my publications, each linking out, plus prominent
  links to my **Google Scholar** and **ORCID** profiles.
- Your Mizzou photos can dress the backdrop.
- Use **Astro content collections**: each publication is a Markdown/data entry;
  the page auto-lists them. Low effort to add new ones later.

---

## 4. Open decisions to resolve along the way

- **Friends "leave a message" backend.** Static GitHub Pages can't store
  submissions on its own. Prefer a **free, no-server** option: a free-tier form
  service (e.g. Formspree free plan) or a guestbook widget. Pick when we reach
  Phase D.
- **Spotify integration depth.** Simple links vs. embedded player widgets vs.
  live "now playing" via the Spotify API (the last needs auth + a backend).
  Embeds are free and need no backend — likely the default.
- **Image pipeline.** Astro's `<Image>` can auto-optimize/resize — worth using
  for the gallery and photo-heavy pages so the site stays fast.
- **Asset sourcing.** Several themes need art you don't have yet (tarot card
  frames, plant illustrations, wall clock, candles). **For now: use
  free-to-use assets** (e.g. public-domain / CC0 sources, free icon/illustration
  libraries). You may **design your own later** to replace them — the themed
  layout won't care which assets fill the slots, so swapping is easy.
- **Interactivity cost.** Default to **free, easily-runnable** approaches:
  vanilla JS + CSS animations, no paid services, no required backend where
  avoidable.

---

## 5. Deployment

- Configure Astro for GitHub Pages: set `site` and `base` in `astro.config.mjs`,
  add the official GitHub Pages Action so pushes auto-build and deploy.
- This replaces serving raw files and ends the local-vs-Pages discrepancy class
  of bugs for good.

---

## Suggested first work session

1. Scaffold Astro, port the global styles into a design system, build
   `Layout.astro` + `Nav.astro`.
2. Fix the image references during the port.
3. Rebuild the Home page (Phase A, minus scroll animation) to validate the
   whole approach end to end.

Hand any phase back to me and I'll implement it.
