# njk.dev

Hugo site, no theme, layouts at project level, one stylesheet. No build
toolchain — no npm, no SCSS, no Hugo Pipes — and Hugo *extended* is not
required.

Everything below was verified against the code on 24 July 2026. Where a line
is a taste decision rather than a fact about the build, it says so.

## Running it

```
hugo server                          # preview; toggle modes via the footer moon/sun
hugo --destination /tmp/njk-build    # verify a clean build
```

`brew install hugo` if it isn't present. The site needs ≥0.145 for the `build:`
front-matter key. CI pins **0.164.0** (see Deploy below), but Homebrew now
ships **0.165.0** — as of 13 August 2026 a local build no longer matches CI.
Nothing warns about the gap; it has caused no trouble so far, but check the
pin before blaming a local/deployed discrepancy on anything else.

**Deploy:** a Cloudflare Worker named `njk-dev` builds from `origin/main`
(migrated from Pages on 26 July 2026). Workers Builds runs `hugo` on every
push to main — automatic, no other branch triggers anything — and serves
`public/` as static assets per `wrangler.jsonc`. That config is deliberately
minimal: assets-only, no script, no bindings; its `not_found_handling` line
makes the custom 404 explicit, which Pages used to infer from the presence of
`404.html`. `HUGO_VERSION` is pinned to **0.164.0** as a build environment
variable in the Workers dashboard. The dashboard holds that value, not this
repo — if a local build ever disagrees with the deployed site, check the pin
first. `public/` and `resources/` are gitignored; deploys build fresh.

The site answers only at `njk.dev`. The `workers.dev` route is disabled on
purpose, and `www` is a proxied dummy A record (`192.0.2.1`) whose zone-level
redirect rule 301s to the apex — DNS and rule live in the Cloudflare zone,
nothing in this repo.

Verified 24 July 2026 on 0.164.0: builds clean, the essay renders, and it stays
out of `sitemap.xml` and `writing/index.xml` as intended.

## Things that will fail the build

Hard `errorf` guards, not preferences. A small edit can trip them.

- **`folio`** (`layouts/writing/single.html:56-63`) — a *listed* post needs a
  folio that is an arabic number. Missing or non-numeric (`folio: iii`) is a
  build error. Unlisted posts skip the check. A missing `date` only warns.
- **`plate`** (`layouts/shortcodes/plate.html:9-14`) — `src`, `caption` and `n`
  are all required; `frame` accepts only `portrait`.
- **`music` / `video`** — same shape: caption and number required, and they
  reject embed URLs in favour of ordinary page URLs.

## Front matter

- **`genre`, not `type`.** Hugo reserves `type` — it changes layout lookup.
  The kicker reads `.Params.genre`, defaulting to `essay` (`single.html:68`).
- **Unlisted posts need both flags:** `unlisted: true` *and* `build:` →
  `list: never` (no underscore — Hugo renamed `_build:` in 0.145). Together
  they drive `noindex`, a suppressed canonical, the `unpaginated` folio, the
  `· unlisted` kicker, no prev/next, and the closing line. To list a post
  later, remove both flags **and** assign a folio, or the build errors.
- `hugo new content writing/a-title.md` scaffolds from `archetypes/writing.md`.
- The `:: now` line on the home page is front matter in `content/_index.md`;
  the layout renders it, so a markdown body there will not appear.

## CSS

`static/css/style.css` is the only stylesheet.

- The **token block** near the top is copied verbatim from the design
  handoff's `tokens.css`. Editing values there puts the site out of sync with
  that source. `--font-serif` (`:68-72`) is a deliberate local addition
  sitting *outside* the block, with a comment saying so — it is not a handoff
  token and shouldn't be folded in.
- **`.prose` is serif; the apparatus is mono.** Anything apparatus-like added
  inside `.prose` needs `font-family: var(--font-mono)` re-asserted — see the
  existing cases at `:492` (h2), `:523` (cite), `:541` (plate), `:665`
  (footnotes).
- **Accent colour flows from `--acc`.** `.page-writing` sets it to violet
  (`:416`); mark tint, hovers and focus rings follow automatically. Hand-setting
  a per-element accent is how that comes apart.

## Wiring worth knowing

- `head-common` is included **before** `mode-init` in all four templates —
  mode-init mutates the theme-color meta that head-common emits.
- `music-player.html` is included only via `{{ if .HasShortcode "music" }}`
  (`single.html:106`). A new single template would need to repeat that, or
  music plates degrade to plain provider links.
- **`music` and `video` are dormant** — complete, guarded, with live CSS, but
  no current essay uses them. `plate` is the only shortcode in use (4 calls in
  the one essay).
- The feed is a project-level `layouts/_default/rss.xml` replacing Hugo's
  embedded template, which ships an invalid empty `<lastBuildDate/>`. It
  currently renders an item-empty channel because the only post is unlisted
  and so excluded from `.RegularPages` — that is expected, and the `with`
  guard is what keeps the empty case valid XML.
- Goldmark: `unsafe = true`, block attributes on, and **typographer off on
  purpose** — the design sets straight quotes throughout; enabling it silently
  curls every apostrophe in the prose.
- `static/favicon.ico` is referenced by no markup. It is served at
  `/favicon.ico`, which is the path legacy agents request implicitly.

## Held

Three `set in berkeley mono · handmade` colophon lines are commented out
(`index.html:78`, `writing/single.html:99`, `writing/list.html:57`) pending a
web font licence. `--font-mono` already leads with Berkeley and degrades
correctly; the `.colophon-note` CSS exists, unused.

## Design source

The original handoff — `BUILD BRIEF.md`, `njk Brand Book.dc.html`,
`tokens.css` — is not stored anywhere this project points to; there is no
archive to fetch, so don't go looking. Its residue in the code (the token
block, straight quotes, the mono/serif split) is documented above and stands
on its own. **A revised style book is in progress.** Treat handoff-derived
detail as background rather than current instruction; where a request
conflicts with it, ask instead of assuming the older document wins — njk has
the final say. The narrative of how the site was built through July 2026 is
archived at `~/Developer/review/njk.dev/` if it's ever needed.
