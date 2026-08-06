# energy-tailgate-theme

Chevron-branded Podigee blog theme for The Energy Tailgate podcast. Compiled from the approved
hi-fi designs (Home, Episode, All episodes, About) into Podigee Liquid templates.

## Deploying via Podigee's Git import

Podigee can pull this theme straight from a Git repo
([docs](https://help.podigee.com/article/160-importing-themes-from-git-repos)). Four constraints
shape how this repo has to be set up:

1. **The repo must be public.** Podigee requires it to be *"publicly readable through an HTTPS
   URL"* — use the HTTPS clone URL, not SSH. There is no deploy-key or token option.
2. **Only HTML and CSS inside `files/` are imported.** *"Any other content will be ignored"* —
   so fonts and images never arrive through the import (see *Fonts* below).
3. **Hard limit of 50 files.** This theme has 9, so there is plenty of headroom.
4. **Re-importing wipes the theme first**: *"all the files of the theme will get deleted and
   replaced by the ones in your repository."* The import is manual — pushing to the repo does
   not auto-deploy — so anything hand-edited in Podigee's theme editor is lost on the next
   import. Treat this repo as the source of truth and never edit templates in Podigee directly.

A branch can be chosen at import time (`master` is Podigee's default; this repo uses `main`).

### Before making this repo public

The licensed Gotham fonts were committed in `f9f844b` and remain in git history even if deleted
in a later commit, so flipping this repository to public would expose the font binaries. Pick
one of:

- **Publish a fonts-free repo** (recommended): a fresh repo containing only `files/*.html` and
  `files/application.css`, with no font blobs in its history. Everything in it is markup and CSS
  that the live site serves publicly anyway, so nothing confidential is exposed.
- **Purge history** in place (`git filter-repo --path files/fonts --invert-paths`) and force-push
  before switching visibility.
- **Skip the Git import** and paste the nine files into Podigee's theme editor by hand. Slower,
  but the repo stays private. Viable because the theme changes rarely.

## Files

| File | Purpose |
|---|---|
| `files/layout.html` | Shared chrome: green utility bar, two-tier Chevron header (inline logo SVG + stacked wordmark), Episodes / Where-to-Listen / Search dropdown panels, dark-blue footer, all theme JavaScript |
| `files/index.html` | Home: cinematic Chevron-hosted video hero, parallax motion control, docked Podigee latest-episode player, More-episodes rows, Where-to-listen band |
| `files/show.html` | Episode detail: breadcrumb, dark-blue title band, docked player, show notes, Listen-on + Share cards (no MP3 download button, per decision) |
| `files/archive.html` | All episodes: live search + topic-chip filtering, season list, pagination |
| `files/about.html` | About: dark-blue hero, premise + host card, three-up strip, CTA band |
| `files/imprint.html`, `files/privacy.html` | Legal pages |
| `files/application.css` | Full stylesheet: Chevron tokens, Gotham @font-face, all components |

The licensed Gotham files are **not** in this repo — see *Fonts* below.

## How dynamic features work

- **Homepage player**: `episode.player` is not available in Podigee's `index.html`, so the
  latest episode is rendered with Podigee's documented iframe endpoint at
  `{{episode.url}}/embed`. This preserves the native player, analytics delivery, and current
  episode data while allowing the player to sit in the cinematic dock.
- **Search overlay** (all pages): does **not** use `podcast.search_box` (that renders Podigee's
  own sidebar widget). Instead it fetches the podcast's RSS feed (`podcast.feeds.mp3`, falling
  back to the blog-relative `/feed/mp3`) client-side and searches title, guest line, description,
  and keywords. Grouped Episodes / Pages results, empty state, Enter forwards to the archive with `?q=`.
- **Episodes menu — "Browse episodes"** (all pages): the dropdown's third column is a scrollable
  episode browser fed by the same client-side RSS index (the `episodes` list variable is only
  documented for `index.html`, so the menu can't render it with Liquid). Each row (eyebrow,
  title, guest line) links to that episode's page; the column hides itself if the feed can't
  be loaded, so the menu degrades to the two static columns.
- **Chapters**: the theme deliberately renders **no chapter list of its own**. The Podigee player
  already ships a chapters panel — set marks in Episode Settings → Chapter Marks and the player
  shows them with click-to-jump seeking, live highlighting of the current chapter, and optional
  per-chapter images and links. A static theme-side list would duplicate that panel a few hundred
  pixels away, without the ability to seek, and could disagree with it (the player reads Podigee's
  episode data, while the feed omits chapters below three marks). The design comp's "In this
  episode" card predates confirming the player does this natively.
- **"More episodes" card** (episode page, bottom of the right rail): up to four other episodes
  from the same RSS index, each row (play glyph, eyebrow, title) linking straight to that
  episode's page, plus an "All episodes" link. The episode being viewed is excluded by URL,
  and the whole card hides itself if the feed can't be loaded or there is nothing else to link.
- **Topic filtering** (archive): chips filter on each episode's **keywords**. Set keywords in
  Podigee per episode using the topic names: `Leadership`, `Technology and innovation`,
  `Fuels and products`. The first keyword renders as the row's topic pill. Deep links work:
  `/archive?topic=Leadership`, `/archive?q=wirth`.
- **Guest line**: the episode **subtitle** field renders as the guest line
  ("With Mike Wirth, chairman and CEO") on the title band, rows, and search results.
- **Episode numbers**: rendered when Podigee exposes `episode.number`; otherwise eyebrows
  fall back to the published month/year. Single digits are zero-padded ("Episode 01") on the
  episode page to match the approved design.
- **Guest bio** (episode page): paste it at the end of the episode's **show notes** in Podigee
  using the theme's classes — `<div class="etg-guest"> <img …> <div class="etg-guest__bio">
  <span class="etg-eyebrow etg-eyebrow--sm">This episode's guest</span> … </div> </div>` —
  and it renders as the divided guest row from the design.

## Variable-availability strategy

Podigee's docs scope some variables to specific templates (`blog.links.*` and `page_content`
to layout, `podcast.description` / `podcast.feeds.*` to sidebar). The theme is written so every
out-of-scope or undocumented use degrades gracefully:

- **Cross-page links** in page templates carry `data-link` attributes; if Liquid renders the
  `blog.links.*` value blank there, the layout script fills the href from a config object
  rendered in layout.html (where those variables are documented).
- **Feed links** carry `data-feed-link` and fall back to the blog-relative `/feed/mp3`.
- **`podcast.description`** on home/about has the approved show description hard-coded as a
  Liquid `{% else %}` fallback.
- **`episode.published_at` / `episode.number` / `episode.keywords` / `episode.transcript`** are
  all `{% if %}`-guarded; eyebrows, pills, and the transcript link simply don't render when a
  variable isn't available in that template.
- **Topic chips** auto-hide (and `?topic=` deep links are ignored) when no episode row carries
  keywords, so the archive never shows a confusing empty filter state.

## GA4 analytics hooks

The theme fires GA4 events when `gtag.js` is present on the blog; without it every call is a
silent no-op, so the theme is safe to ship before analytics is installed. Events:

- `search` — debounced (~1s after typing stops) from both the header search overlay and the
  archive filter box. Params: `search_term`, `search_location` (`overlay` | `archive`),
  `zero_results` (boolean — surfaces queries your content doesn't answer).
- `search_result_click` — a result chosen in the overlay. Params: `search_term`, `result_title`.
- `menu_episode_click` — an episode chosen in the Episodes menu's browse column. Param: `episode_title`.
- `sidebar_episode_click` — an episode chosen in the episode page's "More episodes" card. Param: `episode_title`.
- `filter_topic` — a topic chip selected on the archive. Param: `topic`.
- `share` — X / LinkedIn / Facebook / copy-link on episode pages. Params: `method`, `content_id`.

## Preview gate

The theme currently includes a shared-password preview gate in `layout.html`, so it covers every
blog route. Successful access is remembered in `sessionStorage` for that browser tab session;
adding `?lock=1` clears the session and forces the gate for testing. Background video and audio are
paused while the gate is locked.

This is intentionally a **client-side review deterrent**, not server-side authentication. The
repository is public by requirement, so a determined visitor can inspect or bypass the gate. Use
Podigee's protected-podcast features or an authenticated hosting layer if access control becomes a
production requirement.

Enter in the overlay navigates to `/archive?q=…`; GA4 Enhanced Measurement captures that
automatically as `view_search_results` (`q` is a default site-search param — add `topic` to the
site-search parameter list in GA4 settings to capture topic deep links too). Live typing is
covered by the `search` events above. Mark `search_term`, `search_location`, `zero_results`,
and `topic` as custom dimensions in GA4 to report on them.

## Content the theme reads from Podigee

`podcast.title / subtitle / description / cover_image_url / feeds / copyright / subscribe_button`,
`blog.header_image` (home hero photo; a Dark Blue flat renders without one), `blog.announcement`,
`episode.title / subtitle / description / show_notes / published_at / url / cover_image_url /
player / transcript / keywords`, `pagination`.

## Fonts (licensed Gotham)

The five Gotham cuts are **deliberately not in this repository**. Podigee's Git import only
takes HTML and CSS, so they could never arrive that way — and this repo is public, so shipping
licensed binaries in it would expose them.

`application.css` declares them as WOFF2 at a relative `fonts/` path, which resolves if the
files sit beside the stylesheet in the theme. To serve them from a Chevron host or CDN instead,
one substitution repoints all five:

```
sed -i '' "s|url('fonts/|url('https://your-host/fonts/|g" files/application.css
```

**If that host is a different origin from the blog it must send
`Access-Control-Allow-Origin`.** Browsers refuse cross-origin fonts without it and fall back
silently. This is exactly why chevron.com's own Gotham files cannot be reused: they are
self-hosted WOFF2 under `/assets/fonts/monospace/` and return 200 even cross-site, but carry no
CORS header. Reusing them would need chevron.com to add that header — a one-line change on their
side, and the tidier long-term option if the web team is willing.

Note chevron.com also has **no Gotham Medium**, which this theme uses at weight 500 for the
utility bar; point 500 at Book if you ever switch to their file set.

The files to host (WOFF2, converted from the licensed OTFs — 698 KB down to 167 KB):

```
Gotham_Book.woff2  Gotham_Medium.woff2  Gotham_Bold.woff2
Gotham_Black.woff2  Gotham_Narrow_Bold.woff2
```

Until they are hosted, the fallback stack (Helvetica Neue / Arial) renders everywhere. Nothing
breaks; the display type simply isn't Gotham. Confirm the Gotham licence covers serving them
from whichever host you pick.

## Hand-edited spots to revisit

- `files/archive.html`: season line is static ("Season 1 · 2026") — update per season.
- Platform links (YouTube / Spotify / Apple) are the Chevron channel URLs used in the designs —
  replace with the show's direct listing URLs once live.
- `files/about.html`: host portrait is a styled placeholder (`data-host-photo`) — replace with a
  hosted image URL when available. Host bio is static copy.

## Carried over from the Payload prototype

A Payload CMS + Next.js build of the same designs was explored and then set aside in favour of
this theme. Two artefacts from it are worth keeping:

- **Responsive fixes**, already merged into `files/application.css`: a `≤640px` tier the theme
  previously lacked (the lockup crowded out the Subscribe button below that width), a fix for
  episode rows that squeezed their text column to ~190px and made rows ~590px tall on phones,
  and `z-index` on `.etg-panel` so dropdown clicks are not absorbed by the scrim.
- **A Chevron skin for the Podigee player** (`public/player-theme/` in that project): the stock
  theme restyled to the design system — compact blue CTA, blue Gotham chapter timecodes,
  uppercase tab eyebrows. It is applied through Podigee's *Web Player → External theme* setting,
  which points at hosted `themeCss` / `themeHtml` URLs, so it works with this theme too and has
  the same hosting requirement as the fonts. Podigee sells that as a "Custom webplayer package",
  so confirm entitlement first.

The prototype remains on disk at `~/Desktop/energy-tailgate-site` if the editorial requirements
ever grow into needing a CMS.
