# njk.dev — where we stand

> Catch-up brief for the next Claude (or human) touching this repo.
> Written 15 July 2026, after landing the v2 "writing room"; updated the same
> night for v3 (media plates & apparatus).
> The CURRENT design handoff lives at `~/Downloads/handoff 2/` (v3 — supersedes
> `~/Downloads/handoff/`) — `BUILD BRIEF.md` is the spec, `njk Brand Book.dc.html`
> is the law, `tokens.css` is pinned (unchanged v2→v3). If the handoff folder
> has moved, ask the owner before guessing at values.

## what this is

Hugo static site, no theme, no toolchain beyond Hugo (built on 0.164). Layouts
at project level. One stylesheet (`static/css/style.css`) whose token block is
copied verbatim from the pinned `tokens.css`. Dark is default; light rides
`[data-mode="light"]` on `<html>`, OS-driven with a manual override in
`localStorage` (choosing the OS's own mode clears the override). The only
JavaScript is inline: the roll, the mode init, the day-line toggle, one console
line per page.

The site is a bound book. Folios: `fol. 01` home · `fol. 02` the writing index
· `fol. 03+` posts in publication order, assigned by hand, never reused ·
`fol. ∅` the 404 · unlisted posts read `unpaginated`.

## the rooms

- `/` — the landing. Each load rolls one of six tempers (gold/orange/teal/
  violet/green/gray) before paint and sets `--acc`; the footer names the day.
  No persistence — the loop is the point.
- `/writing/` — the writing room, new in v2. Semantic, not rolled: always
  violet · reading. No roll script on typed pages.
- `/404.html` — always gray · log. Does not roll. `noindex`.

## v2: what was added and how

### templates
- `layouts/writing/single.html` — the post. Breadcrumb `njk.dev / writing`
  (leaf violet always; a SPAN, not a link, per the design — "-> all writing"
  is the way back, and unlisted leaves point nowhere), folio from front
  matter, kicker `:: {genre} · {n} min` (`.ReadingTime`), 32px/300 title,
  dateline `dd · roman-month · yyyy · {occasion}` (roman month computed from
  `.Date.Month` in the template), 44px stub, prose, `:: further` prev/next
  (listed posts only — note Hugo's `.PrevInSection` = the OLDER post; verified
  empirically), violet footer mark (`title="he read it first"`), violet
  console line. The folio is GUARDED: a listed post with a missing or
  non-numeric folio is a build error with a ledger-rule message (the ledger is
  arabic, zero-padded — `folio: 3` renders `fol. 03`; `folio: iii` refuses to
  build); a missing date warns at build time.
- `layouts/writing/list.html` — the index at `/writing/`, fol. 02: `:: writing`
  ledger of arrow links with datelines, newest first.
- `layouts/shortcodes/plate.html` — `{{</* plate src="..." caption="..." n="i" */>}}`.
  Hard-errors (`errorf`) on missing src, caption, or number: no image without
  a word. Numbers are roman, per essay, by hand.
- Unlisted posts: front matter `unlisted: true` **plus** `build: { list: never }`.
  (The brief says `_build:` — Hugo ≥0.145 renamed it to `build:`; the old
  spelling is a build error on 0.164.) Gets: `noindex`, no canonical, folio
  `unpaginated`, kicker `· unlisted` (replaces the minute count), no
  prev/next, closing line `shared by hand · no index points here`, and it's
  out of the index/RSS/sitemap via `list: never`.

### repo cleanups (brief §10 — done)
- Mode/day-line JS deduped into partials, one copy each (was pasted verbatim
  per page): `layouts/partials/head-common.html` (metas, favicons, font
  preloads, stylesheet), `mode-init.html` (before-paint mode script — include
  AFTER head-common, it needs the theme-color meta to exist), `day-line.html`
  (the moon/sun sentence + paint/toggle script; takes `before`/`after` HTML
  fragments to shape each page's sentence).
- Theme-color flash fixed: `mode-init` now sets `<meta name="theme-color">` to
  `#f3f1ea` when it applies light, so light-mode arrivals don't get dark
  browser chrome before the footer script runs.
- Prism is canon: n gold · j orange · k green (creed order). Comments updated;
  the old "j teal" note is superseded.
- Token block synced with pinned v2 `tokens.css`: added the writing-room vars
  (`--fs-title`, `--ink-soft`, `--ink-dim`, `--plate`) in both modes.

### config (`hugo.toml`)
- `disableKinds` relaxed from v1 (which disabled section/page/rss) to
  `['taxonomy', 'term']`.
- `[outputs]`: home HTML only; sections HTML + RSS → the feed lives at
  `/writing/index.xml` and only there. The feed uses a project-level
  `layouts/_default/rss.xml` (NOT Hugo's embedded one) because the embedded
  template ships an invalid empty `<lastBuildDate/>` while the ledger has no
  listed posts. Both writing templates advertise it via
  `<link rel="alternate" type="application/rss+xml">`.
- Goldmark: `unsafe = true` (own hand-authored prose; needed for `<cite>` in
  pulled quotes and the one accent `<span>` per essay) and block attributes
  (`{.preface}` marks the opening italic paragraph).
- Typographer DISABLED — the designs render straight quotes throughout (a
  typed page, not a typeset one); without this, Goldmark silently curls every
  apostrophe and quote in the prose.

### fonts
- Self-hosted IBM Plex Mono 300/400/600 **plus new Italic 400**
  (`static/fonts/IBMPlexMono-Italic.woff2`, Fontsource latin subset — the
  writing room uses true italics: preface, pulled quote, cited titles). Italic
  is preloaded on post pages only.

### css (`static/css/style.css`, "THE WRITING ROOM" section)
- `.page-writing` sets `--acc: var(--t-violet)` — the accent-driven furniture
  (mark tint, hovers, focus rings) follows automatically; that's why the post
  templates don't need a `.mark-violet` class.
- Prose is styled against Goldmark's actual output: `##` section markers are
  `h2`s (the literal `##` ships via `::before`), footnotes restyled from
  Goldmark's `.footnotes` div (default `<hr>` hidden, `:: notes` label via
  `::before`, violet numbers via CSS counter in an auto-width grid column with
  the design's 14px gap, `↩` back-link glyph replaced with `<-` via font-size
  0 + `::after`). A11y over the markup we don't control: the post's inline
  script sets `aria-label="back to reference n"` on each back-link (the ↩
  glyph would otherwise be the accessible name), and the generated `:: notes`
  / `<-` content carries CSS alt-text (`content: ... / "..."`, with a plain
  fallback declaration for older engines). The mode toggle keeps a stable
  accessible name (`aria-label="light mode"`, state on `aria-checked`); the
  moon/sun word is presentation.

## v3: media plates & apparatus (brief §9 "musical examples & video")

Numbering is by ROLE, each its own hand-assigned roman count: images `pl.` ·
music `ex.` · video `fig.` (a chant recording on YouTube is still `ex.`).
Reference design: `njk Media Plates.dc.html` — the CHOOSER pattern is law.
(The updated `njk Post.dc.html` shows an older single-provider "-> listen"
button variant; the brief explicitly names Media Plates as the reference and
the Brand Book's apparatus line agrees — the button variant is superseded.
Also: the reference's Spotify/YouTube example URLs are rickroll placeholders,
not real citations; only the Apple *Te lucis* URL is real.)

- `layouts/shortcodes/music.html` — one plate per piece, any subset of
  apple/spotify/youtube. Resting state: title (+ faint `· subtitle`) + work
  line. NO iframe, NO third-party request before a click. The caption is the
  control surface: `{caption} — -> listen via apple music · spotify ·
  youtube`; each `· provider` pair is nowrap so a wrap never orphans a
  separator; single provider drops the separators. Provider words are ANCHORS
  to the provider pages (no-JS fallback), upgraded to players by JS. Guards
  errorf on: missing title/caption/n, no provider at all, and pasted EMBED
  urls (it wants the page urls).
- `layouts/partials/music-player.html` — inline JS, included by single.html
  ONLY when `.HasShortcode "music"` (video needs no JS). Click loads that
  provider's iframe; the same words switch players afterward. Embed
  derivations (heights/aspect always authored on our side): apple →
  `embed.music.apple.com` + `&theme=dark|light` from the mode at click time,
  150 track (`?i=`/`/song/`) / 450 album, radius 10; spotify →
  `/embed/`, query stripped, 152 track / 352 album+playlist, radius 12;
  youtube → `youtube-nocookie.com/embed/{id}`, 16:9, radius 10. The chosen
  door is stored in `localStorage["njk-listen"]`; on later plates/visits the
  usual door's word wears the temper (`--acc`, i.e. the PAGE's violet — never
  a provider color) but NEVER auto-loads. Behavior is regression-tested: the
  emitted script was executed against a stub DOM in JavaScriptCore
  (derivations, switching, storage, return-visit paint, no-autoload).
- `layouts/shortcodes/video.html` — video that IS the content: direct
  youtube-nocookie iframe (loading=lazy), 16:9, `fig. {n}`, caption required,
  `title` required (it names the frame for screen readers). Accepts a watch/
  youtu.be/shorts/embed url or a bare id; errorf otherwise.
- CSS: `.plate` grew `overflow: hidden` (the guard); `.plate-music` /
  `.plate-video` sections at the end of the writing room. `.listen-link`
  rules are scoped `.plate-music .listen-link` so they outrank `.prose a`.
- The placeholder essay carried `ex. i` (*Te lucis ante terminum*, apple
  only — the one real URL) as the live demo; it left with the placeholder
  (deleted 18 july 2026). No current essay ships a music plate — the
  chooser is dormant but stays regression-tested.

## v4: the charbel essay & the twin title (18 july 2026)

- The placeholder is gone; the real essay landed:
  `content/writing/seen-in-secret.md` — "everyone sees it. that's the
  design." Same publish pattern as the placeholder had: `date` 2026-07-22
  (the feast — the dateline), `publishDate` 2026-07-18 (a plain build
  includes it now), unlisted (both flags), shared by hand.
- **THE TWIN TITLE.** The h1 differs by mode: the dark page reads "nobody
  sees it. that's the design."; the light page strikes that claim through
  and answers "everyone sees it. that's the design." Mechanism: optional
  front matter `titleDark` (plain string) + `titleLight` (may carry `<s>`,
  shipped via `safeHTML`); `single.html` renders one h1 with two
  aria-hidden spans (`.title-hidden` / `.title-seen`) and puts the
  canonical `title` on the h1's `aria-label`. The swap is CSS-only, keyed
  to `[data-mode="light"]` — NOT `prefers-color-scheme`, which would
  ignore the manual toggle; dark is the attribute-less default, so the
  hidden span leads. `@media print` shows the struck version (paper reads
  in the light). `title` stays the ONE canonical string everywhere
  outside the rendered h1: tab, og/twitter, feed, screen readers. Posts
  without `titleDark` render exactly as before.
- The law of the feature: never announced, never explained — no toggle
  hint, no tooltip, no footnote about it. A reader who flips the mode
  finds it alone. (Origin + full testing checklist: the owner's
  `charbel/title-design-spec.md`, written against a light-default
  prefers-color-scheme site — adapted here to the `[data-mode]` cascade.)
- Known, accepted: browser reader modes strip CSS and may show both spans
  concatenated. The spec's hardening (dark variant via `::before` content)
  was deliberately NOT taken unless it proves a real problem — it costs
  find-in-page and copy/paste.
- Four plates at `static/plates/` (recompressed from the owner's originals
  in `charbel/`, which stays untracked): `annaya-tomb-leo-xiv.jpg` (pl. i),
  `hermitage-cell.jpg` (pl. ii), `charbel-portrait.jpg` (pl. iii),
  `annaya-pilgrimage-road.jpg` (pl. iv).
- `plate.html` grew an optional `frame="portrait"` (any other value is a
  build error): the image hangs matted — natural aspect, `max-height`
  420px, centered on the recessed ground — instead of full-bleed width,
  so tall images don't tower on phones. pl. ii and pl. iii wear it;
  landscape plates are untouched.
- All three footnote URLs verified live 18 july 2026 (two vatican.va; the
  CNA/NCRegister pilgrimage piece is from 24 july 2024).

## flags for right now — read before touching

1. **`set in berkeley mono · handmade` is COMMENTED OUT** (Go template
   comments in `layouts/index.html`, `layouts/writing/single.html`,
   `layouts/writing/list.html`) because Berkeley Mono isn't licensed/shipped
   yet. When the web license lands (owner expects ~August 2026): add the
   `Berkeley Mono Variable form` @font-face ABOVE the Plex ones in style.css
   (the stack in `--font-mono` already degrades correctly — do not
   restructure), then un-comment those three lines. The `.colophon-note` CSS
   already exists and waits.
2. **The first essay is LIVE (unlisted).** `content/writing/seen-in-secret.md`
   — the charbel essay (see v4), published 18 july 2026 via `publishDate`
   (the `date` stays 2026-07-22, the feast, for the dateline; that split is
   how a future-dated leaf builds without `buildFuture`). Unlisted: reachable
   only by its URL (`/writing/seen-in-secret/`), out of index/RSS/sitemap,
   `noindex`, `unpaginated`. It carries the twin title — mind the v4 rules
   before touching the h1 or metadata. The hermit placeholder
   (`the-hermit-who-would-not-stay-hidden.md`) was deleted the same day; it
   lives in git history if ever wanted. If the essay is ever listed, remove
   both unlisted flags and assign the next folio by hand.
3. **Front matter key is `genre`, not `type`** — `type` is reserved by Hugo
   (it changes layout lookup). The brief's kicker `:: {type}` reads from
   `.Params.genre`, default `essay`.
4. **X handle migration done** (`njkdev`, 17 july 2026): link in
   `content/_index.md` updated.
5. `public/` is gitignored — deploys build fresh; don't commit build output.

## don'ts (decided; Brand Book is law)

No second typeface. No boxes/cards/rounded containers. No multicolor wordmark
at rest. No announcing easter eggs. No Ubuntu orange `#E95420`. No animation
beyond the prism transitions. No analytics. At most ONE accent-colored
sentence per essay. Every plate has a caption. Meaning never moves between
modes. Don't edit token values — they're pinned; the handoff's `tokens.css`
is the source.

## the reviews

### v3 media plates (15 july 2026, second round)

One focused review pass over the media-plates work; findings verified, then
the fixed player script was re-executed against the stub-DOM harness (all
assertions green). Fixed: Spotify `/intl-xx/` URLs 404 at the embed path
(segment now stripped in the JS; the reviewer verified the 404 against the
live endpoint); modifier-clicks (cmd/ctrl/shift/alt) on the source words now
keep their browser meaning instead of being hijacked into an inline load;
`geo.music.apple.com` share links now rewrite to the embed host (and apple/
spotify validation is anchored to the real https:// hosts); the music
shortcode's `youtube` param is now validated at build time like video's, and
both accept `/live/` URLs.

Declined deliberately: a mode toggle does NOT retheme an already-loaded Apple
player — repainting means reloading the iframe and cutting off playback; the
next click picks up the right theme. Also noted: if a non-writing single
template ever exists, it must remember the `.HasShortcode "music"` include or
music plates degrade to plain provider links (functional, no chooser).

### v2 writing room (15 july 2026)

Three independent review passes ran before the first push (code correctness,
design fidelity vs the handoff, config/SEO/a11y details); every finding was
re-verified against built output before fixing. Landed: the folio build
guards, `.breadcrumb .crumb-writing` specificity (a bare `.crumb-writing`
loses to `.breadcrumb a`), `.mark-ghost`'s tint moved inside the `@supports`
mask guard, typographer off, the custom RSS template + feed discovery links,
twitter:card + og:image dims and a whitespace-collapsed `.Summary` description
fallback on writing pages, the footnote a11y labels, the stable toggle name,
the design-true footnote gutter, and the plate TODO moved out of the rendered
body.

Reviewed and deliberately NOT taken (don't re-litigate without the owner):
- Console line uses the mode-resolved `--t-violet` (light arrivals get
  `#6d5ad6`); the design hardcodes dark `#948ae3` — the token system is
  considered more correct here.
- No robots.txt (harmless 404) and Hugo's generator meta on the home page —
  left alone.
- `--fs-body` 0.97rem = 15.52px vs the design's 15.5px (and colophon
  10.496 vs 10.5) — the tokens are pinned verbatim; the sub-pixel deltas are
  the tokens' own.
- Default-build warnings that `writing/single.html`/`plate.html` are unused
  are expected while the only post is a future-dated draft; they disappear on
  publish.

## verifying changes

```
hugo --destination /tmp/njk-build --cleanDestinationDir
```

(The first essay publishes via `publishDate`, so a plain build includes it;
use `-D -F` only when working on new drafts.) Check: the post
page's folio/kicker/notes markup, `/writing/index.html` ledger, and that the
unlisted post appears NOWHERE in `sitemap.xml` or `writing/index.xml`. A plain
`hugo` build must stay clean and must NOT emit the draft. `hugo server -D` to
eyeball both modes (toggle via the moon/sun in the footer).
