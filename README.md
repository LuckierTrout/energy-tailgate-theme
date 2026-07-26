# energy-tailgate-theme

Chevron-branded Podigee blog theme for The Energy Tailgate podcast. Compiled from the approved
hi-fi designs (Home, Episode, All episodes, About) into Podigee Liquid templates.

## Files

| File | Purpose |
|---|---|
| `files/layout.html` | Shared chrome: green utility bar, two-tier Chevron header (inline logo SVG + stacked wordmark), Episodes / Where-to-Listen / Search dropdown panels, dark-blue footer, all theme JavaScript |
| `files/index.html` | Home: hero with scrim, docked latest-episode card, More-episodes rows, Where-to-listen band |
| `files/show.html` | Episode detail: breadcrumb, dark-blue title band, docked player, show notes, Listen-on + Share cards (no MP3 download button, per decision) |
| `files/archive.html` | All episodes: live search + topic-chip filtering, season list, pagination |
| `files/about.html` | About: dark-blue hero, premise + host card, three-up strip, CTA band |
| `files/imprint.html`, `files/privacy.html` | Legal pages |
| `files/application.css` | Full stylesheet: Chevron tokens, Gotham @font-face, all components |
| `files/fonts/` | Licensed Gotham cuts (Book, Medium, Bold, Black, Narrow Bold) |

## How dynamic features work

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

`application.css` references `fonts/Gotham_*.otf` relative to itself. If Podigee's theme
uploader does not accept the font files alongside `application.css`, host the `fonts/` folder
on a CDN and search-replace `url('fonts/` with `url('https://your-cdn/fonts/` in the CSS.
Fallback stack is Helvetica Neue / Arial, so nothing breaks without them.

## Hand-edited spots to revisit

- `files/archive.html`: season line is static ("Season 1 · 2026") — update per season.
- Platform links (YouTube / Spotify / Apple) are the Chevron channel URLs used in the designs —
  replace with the show's direct listing URLs once live.
- `files/about.html`: host portrait is a styled placeholder (`data-host-photo`) — replace with a
  hosted image URL when available. Host bio is static copy.
