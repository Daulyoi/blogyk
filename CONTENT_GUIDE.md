# Content & Customization Guide

This site is a [Zola](https://www.getzola.org) static site using the [`serene`](https://github.com/isunjn/serene) theme (cloned as `themes/serene`, branch `latest`).

Theme docs: `themes/serene/USAGE.md` + `themes/serene/zola.toml.example`. This file is the short, site-specific workflow.

## 0. Layout of this repo

```text
zola.toml                  # site config (theme = "serene")
content/
  _index.md                # home page (template home.html)
  posts/_index.md          # main blog section (posts.html + post.html)
  posts/hello-world.md     # sample post — edit/delete me
  about/_index.md          # prose page
  collections/_index.md    # prose page hosting collections
  collections/projects.toml# collection data file
static/                    # copied to / verbatim (put img/, font/, icon/ here)
sass/                      # empty = use theme styles
templates/                 # empty = use theme templates; override here
themes/serene/             # DO NOT edit directly (submodule, will conflict on update)
public/                    # build output (zola build)
```

## 1. Preview and build

Zola `>= 0.23.4` is required (`theme.toml`). You currently don't have `zola` on PATH.

```sh
# if you downloaded the test binary:
 /tmp/opencode/zola --version
 /tmp/opencode/zola check
 /tmp/opencode/zola serve      # http://127.0.0.1:1111, live reload
 /tmp/opencode/zola build      # writes public/

# install permanently (pick one):
cargo install --locked zola
# or download zola-v0.23.6-x86_64-unknown-linux-gnu.tar.gz from
# https://github.com/getzola/zola/releases and put `zola` on PATH
```

`zola check` passes today: `1 pages, 3 sections`.

## 2. First-run checklist (replace placeholders)

All placeholders are still the `zola.toml.example` values (`xxxx`, `Jhon Wick`, `<your-username>`).

In `zola.toml`:

| Field | What to do |
|---|---|
| `base_url` | Your production URL, e.g. `https://yourdomain.com`. Needed for feeds, sitemap, OG links. |
| `title` / `description` | Site title + tagline. Used in `<title>`, SEO, feed. |
| `[extra] name/handle/bio` | Home-page identity. Comment out any line you don't want. |
| `# avatar` | Uncomment + add `static/img/avatar.webp` (webp recommended). |
| `[extra] links` | Replace `url`s. `icon` is an svg basename from `themes/serene/static/icon` (`github`, `email`, `twitter`, `mastodon`...). |
| `footer_copyright` | e.g. `© 2026 Your Name`. |
| `nav` | Currently `posts /collections /about`. Add/remove `{ name, path }` entries; internal paths start with `/`, external URLs open in new tab. |
| `blog_section_path` | Must match your main blog section (`/posts`). Home "recent" list + tag pages use it. |

Then update home body in `content/_index.md` (the text below `+++`).

## 3. Home page — `content/_index.md`

```md
+++
template = 'home.html'
[extra]
lang = 'en'
footer = false        # show footer on home?
recent = true         # show recent posts from blog_section_path?
recent_max = 15
recent_more_text = "more »"
+++

Hi, I'm ...   <- markdown body shown under your name/bio
```

## 4. Writing a post

Create `content/posts/my-slug.md`:

```md
+++
title = "My Post"
description = "One-line summary (SEO + cards)."
date = 2026-10-13
updated = 2026-10-14
draft = false   # true hides it from build (use `zola serve --drafts` to preview)

[taxonomies]
categories = ["general"]
tags = ["hello", "serene"]

[extra]
lang = "en"
toc = true
code_copy = true
comment = false
math = false
mermaid = false
featured = false      # shows * before title in lists
outdated_alert = false
reaction = false
og_image = "cover.png"  # optional, see §7
+++

Your markdown here...
```

Rules:

- Filename becomes slug (`my-slug.md` → `/posts/my-slug/`). Date sorting comes from `date` + `sort_by = "date"` in `posts/_index.md`.
- Delete `content/posts/hello-world.md` when you have a real post.
- Section `_index.md` (`content/posts/_index.md`) controls the list: `title/subtitle`, `categorized = false|true`, `back_to_top`, `generate_feeds = true` (recommended per-section feed at `/posts/feed.xml`).
- Display options cascade, closest wins: **post front matter → section `_index.md` → `[extra]` in `zola.toml`**. Applies to `toc, code_copy, comment, math, mermaid, reaction, outdated_alert`.

## 5. Sections: about, collections, more blogs

- **Prose page** (about): `content/about/_index.md` with `template = "prose.html"`, `[extra] title/subtitle`. Add `{ name = "about", path = "/about" }` to `nav` (already done).
- **Second blog list** (e.g. `notes`): copy `content/posts/_index.md` to `content/notes/_index.md` (keep `template="posts.html"`, `page_template="post.html"`), add to `nav`. Each can have its own feed. `blog_section_path` stays pointing at your *main* one (`/posts`).
- **Collections page**: see §6. Ours is `content/collections/_index.md` (`template="prose.html"`).

## 6. Collections — `content/collections/`

A collection is a `.toml` + a `{{ <collection ... /> }}` call. Example `projects.toml`:

```toml
layout = "card"   # card | row | tile | gallery
# flow = "grid"   # stack | inline | grid (sensible default per layout)

[[item]]
title = "Tokio"
link = "https://example.com"
content = "Markdown-capable description."
tags = ["rust", "async"]
badge = "2025"
featured = true
# subtitle, icon, image, rotate also supported — see themes/serene/USAGE.md §Collection
```

Render it in `_index.md`:

```md
{{ <collection file="projects.toml" section /> }}
```

Multiple collections per page are fine — add more `.toml` files + headings between calls.

## 7. Images and static files

- Anything in `static/` is served at `/`. `static/img/avatar.webp` → `avatar = "img/avatar.webp"`.
- **Favicon**: generate at [favicon.io](https://favicon.io/favicon-converter/), put `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png` in `static/img/`.
- **OG/social image**: `og_image = "img/og.png"` in `zola.toml` (default) or per-post `og_image` in front matter (full URL, `/`-path in `static/`, or colocated asset filename). ~1200×630 recommended.
- **Colocated assets**: put `cover.png` next to `my-post.md` in `content/posts/my-post/cover.png` + `index.md`, then reference by filename. Works for `figure` and `og_image`.

## 8. Markdown extras serene supports

GitHub alerts → styled callouts (5 types):

```md
> [!NOTE]
> note text
> [!TIP]
> tip text
> [!IMPORTANT] / [!WARNING] / [!CAUTION]
```

Custom callout titles (e.g. translation) via CSS vars in your `_custom_css.html`:

```css
--callout-note-title: "注意";
```

Tera components (must be top-level, not inside lists):

```md
{{ <figure src="/img/pic.png" alt="alt" caption="caption [link](https://example.com)" width="600" height="400" /> }}
{{ <figure src="colocated.png" caption="hi" page /> }}

{% <quote cite=""> %}...{% </quote> %}
{% <detail title="More" default_open={false}> %}...{% </detail> %}
{% <mermaid> %}flowchart LR; A-->B{% </mermaid> %}
{{ <youtube id="<video-id>" autoplay={true} /> }}
```

Literal `{{`/`{%` in code samples: wrap with `{% raw %}...{% endraw %}`.

Code blocks support [Zola annotations](https://www.getzola.org/documentation/content/syntax-highlighting/#annotations). Highlighting style must stay `style = "class"` (already set) — serene ships `giallo-light/dark.css`.

## 9. Optional features (all off by default)

| Feature | How to enable |
|---|---|
| `toc` | `toc = true` (default on). Per-post/section override. |
| KaTeX math | `math = true`, then `$...$` / `$$...$$`. |
| Mermaid | `mermaid = true`, then `{% <mermaid> %}` block. |
| Outdated alert | `outdated_alert = true` + `outdated_alert_days = 120` (+ custom `_text_before/after`). |
| Comments (giscus) | Create `templates/_giscus_script.html` with your giscus snippet (`data-theme` → `https://<your-domain>/giscus_light.css`), then `comment = true`. |
| Emoji reactions | Deploy e.g. [isunjn/reaction](https://github.com/isunjn/reaction), set `reaction_endpoint` + `reaction = true`. |
| Analytics | Create `templates/_head_extend.html` with your `<script>` tags. |
| Dark/light | `color_scheme = "auto"` (follows system + toggle) \| `"light"` \| `"dark"` (locked). |
| Feeds | Recommended: `generate_feeds = false` globally + `generate_feeds = true` in list sections → `/posts/feed.xml`. Or global feed via `zola.toml`. `feed_filenames = ["feed.xml"]` uses serene's atom template. |

## 10. Look & feel overrides (theme-safe)

Never edit `themes/serene/` — overrides win automatically by path:

- `templates/_custom_css.html` — copy from theme, change `--primary-color`, fonts, callout titles.
- `templates/_custom_font.html` — create, paste Google Fonts `<link>`s or self-hosted `@font-face` (put woff2 in `static/font/`), then set `--main-font` / `--code-font` in `_custom_css.html`.
- Any `templates/*.html`, `static/icon/*.svg` (set `width/height="18"`, `fill="currentColor"`), `sass/*` — copy same-named file from theme to repo root and edit there.

## 11. Updating serene

```sh
git submodule update --remote themes/serene
# then: re-check CHANGELOG.md for breaking changes,
# re-apply any copied _custom_css.html / template overrides,
# zola check && zola build
```

If you overrode templates/CSS, diff your copies against the new theme versions before rebuilding.
