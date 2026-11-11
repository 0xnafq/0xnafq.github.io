# Changelog

All notable changes to this project will be documented in this file.

## [6.0.0] - 2026-09-06

- feat: migrate to zola v0.23 / Tera v2, the minimum zola version required is now `v0.23.4`, shortcodes are rewritten as Tera components
- feat: tag links are now generated via zola's taxonomy API instead of hardcoded `/tags/` paths, so the `taxonomy_root` config option (e.g. `taxonomy_root = "blog"` for `/blog/tags/xxx` URLs) is respected
- feat: support multiple blog-like list sections — post pages now read config from their own parent section instead of the `blog_section_path` one, and each list section can have its own feed (`generate_feeds = true` in its `_index.md`); `blog_section_path` now only designates the main section used by the home page's recent posts and the tags pages
- refactor: `blog.html` template is renamed to `posts.html`, change `template = "blog.html"` to `template = "posts.html"` in your blog section's `_index.md`
- feat: display options (`toc` / `code_copy` / `comment` / `math` / `mermaid` / `reaction` / `outdated_alert` / `date_format`, etc.) now follow a unified fallback chain: post front-matter → section `_index.md` → `[extra]` of `zola.toml`, the closest one wins — notably `math` / `mermaid` can now be enabled for a whole section, and site-wide defaults can be set in `zola.toml`
- refactor: several options are renamed for clarity: `force_theme` (`false | "light" | "dark"`) → `color_scheme` (`"auto" | "light" | "dark"`), `copy` → `code_copy`, `outdate_alert*` → `outdated_alert*`, and `id` of the home section → `handle`
- refactor: the `sections` config option is renamed to `nav`, and its `is_external` field is removed — a `path` starting with `/` is an internal link, anything else (e.g. an `https://` URL) is treated as an external link automatically; external nav links now show a `ne-resize` cursor
- refactor: `name` / `handle` / `bio` / `avatar` / `links` of the home page are moved from the home section's `_index.md` to `[extra]` of `zola.toml`
- refactor: the example config file is renamed from `config.example.toml` to `zola.toml.example`, following zola v0.23's new default config file name `zola.toml` (the old `config.toml` name still works)
- refactor: the callout components (`note` / `tip` / `important` / `warning` / `caution`) are removed in favor of zola's native [GitHub alert syntax](https://github.com/orgs/community/discussions/16925) (`> [!NOTE]`), styled with icon and title by the theme; the title texts can be customized via css variables (`--callout-note-title`, etc.)
- feat: collections are redesigned — item appearance (`layout = "card" | "row" | "tile" | "gallery"`) and arrangement (`flow = "stack" | "inline" | "grid"`, the grid adapts its column count to the available width) are now declared at the collection level, and all layouts share one unified set of item fields (`title` / `subtitle` / `content` / `icon` / `image` / `link` / `badge` / `tags` / `featured`) with graceful degradation; item `link`s are auto-detected as internal or external, and `image` supports section-colocated files; `card` items can additionally show a picture on the left via `image` (the small `icon` before the title is a separate field)
- chore: upgrade Mermaid to 11.17.2 and KaTeX to 0.18.6, refreshing KaTeX resource integrity hashes

### Migrate from zola v0.22

Zola v0.23 removed shortcodes and Tera macros in favor of [Tera components](https://www.getzola.org/documentation/content/overview/#templating-your-content), and your markdown content is now itself a Tera template. Once you update zola and this theme, you need to update your content accordingly:

- Shortcode calls in your markdown must be rewritten in component syntax. Arguments other than strings are wrapped in `{...}`:
  - `{{ figure(src="...", width="600") }}` → `{{ <figure src="..." width="600" /> }}`
  - `{% quote(cite="...") %} ... {% end %}` → `{% <quote cite="..."> %} ... {% </quote> %}`
  - `{{ youtube(id="...", autoplay=true) }}` → `{{ <youtube id="..." autoplay={true} /> }}`
  - `{{ collection(file="projects.toml") }}` → `{{ <collection file="projects.toml" section /> }}` (note the extra `section`)
- The callout shortcodes are removed, rewrite them with the GitHub alert syntax (custom titles are not supported, the title is always the type name):

  ```md
  {% note(title="Note") %}
  note text
  {% end %}
  ```

  becomes:

  ```md
  > [!NOTE]
  > note text
  ```

- For `figure` with a colocated image, pass the page context: `{{ <figure src="img.png" page /> }}` (or `section` instead of `page` when used in a section's `_index.md`)
- `width` / `height` of `figure` must be strings: `width="600"`, not `width=600`
- Literal `{{` or `{%` in your content (e.g. code blocks showing template syntax, GitHub Actions `${{ ... }}`) must be wrapped with `{% raw %}` ... `{% endraw %}`, otherwise zola will try to interpret them as Tera syntax. For files full of such content, you can instead list them in the `skip_content_templating` config option (glob patterns) to opt them out of templating entirely (components won't work in those files)
- Component calls must be at the top level of your markdown content — nesting them inside a list item is no longer supported (the component's HTML output breaks the list's indentation rules and produces broken HTML). If you had e.g. a callout inside a list item, move it out of the list
- The `date` filter is now backed by [jiff](https://docs.rs/jiff/latest/jiff/fmt/strtime/index.html) instead of chrono. Common `date_format` values like `"%b %-d, %Y"` still work, but chrono-specific specifiers (e.g. `%+`) are no longer supported and will fail the build — check your `date_format` options against the jiff docs
- Syntax highlighting CSS files (`giallo-light.css` / `giallo-dark.css`) are now generated directly into the output directory instead of `static/`, so you can remove them from your `static/` folder and `.gitignore`
- If you have custom templates that override serene's, they must be migrated to Tera v2 as well, see the [Tera migration guide](https://github.com/Keats/tera/blob/master/MIGRATION.md)
- The `blog.html` template is renamed to `posts.html`: change `template = "blog.html"` to `template = "posts.html"` in your blog section's `_index.md`
- Zola's default config file name is now `zola.toml` — you can rename your `config.toml` to `zola.toml` (optional, the old name still works)
- Several options are renamed, update them in your `zola.toml` / section `_index.md` / post front-matter:
  - `force_theme = false | "light" | "dark"` → `color_scheme = "auto" | "light" | "dark"` (`false` becomes `"auto"`)
  - `copy` → `code_copy`
  - `outdate_alert` / `outdate_alert_days` / `outdate_alert_text_before` / `outdate_alert_text_after` → `outdated_alert` / `outdated_alert_days` / `outdated_alert_text_before` / `outdated_alert_text_after`
  - `id` in the home section's `_index.md` → `handle`
  - `sections` in `zola.toml` → `nav`, and remove the `is_external` field from its entries (external links are now detected automatically from the `path`)
  - `name` / `handle` / `bio` / `avatar` / `links` move from the home section's `_index.md` to `[extra]` of `zola.toml`
- Collection toml files use a new schema, update them as follows:
  - each `[[collection]]` becomes `[[item]]`, and the per-item `type` moves to a single collection-level `layout` at the top of the file: `card` → `layout = "card"`, `card_simple` → `layout = "row"` plus `flow = "stack"` (rename its `content` to `subtitle`), `entry` → `layout = "row"`, `box` → `layout = "tile"`, `art` → `layout = "gallery"`, `art_simple` → `layout = "gallery"` plus `flow = "grid"`
  - rename item fields: `img` → `image` (`icon` keeps its name), and both `date` and `footer` → `badge` (a short mark rendered as-is: a year, a date range, a rating, a status...)
  - remove `type = "br"` items — arrangement is now controlled by the collection-level `flow` (`"stack"` / `"inline"` / `"grid"`) option

## [5.7.0] - 2026-08-09

- feat: add open graph & twitter card meta tags and canonical link
- fix: resolve colocated figure image URLs [@uchouT](https://github.com/uchouT) ([#109](https://github.com/isunjn/serene/pull/109))
- fix: art collection content margin

## [5.6.3] - 2026-02-14

- fix: back link behavior on new tab

## [5.6.2] - 2026-02-13

- feat: add markdown support for collection's content and subtitle [@paulhdk](https://github.com/paulhdk) ([#104](https://github.com/isunjn/serene/pull/104))

## [5.6.1] - 2026-02-01

- fix: codeblock with filename

## [5.6.0] - 2026-02-01

- feat: adapt zola v0.22 new code highlighting, custom highlight themes (`serene-light`/`serene-dark`) are removed in favor of built-in themes
- feat: add `github_alerts` color styling (zola v0.21+ feature)
- feat: add `rel_me` option for links (useful for Mastodon verification)
- feat: add `youtube` shortcode [@gloomydumber](https://github.com/gloomydumber) ([#92](https://github.com/isunjn/serene/pull/92))
- fix: input element color scheme should be changed along with light/dark mode change

### Migrate from zola v0.21

Since this version, the minimum zola version required has changed to `v0.22.1`.

Replace the old `[markdown]` section in your `config.toml`:

```toml
# Old (remove these)
[markdown]
highlight_code = true
highlight_theme = "css"
extra_syntaxes_and_themes = ["themes/serene/highlight_themes"]
highlight_themes_css = [
  { theme = "serene-light", filename = "hl-light.css" },
  { theme = "serene-dark", filename = "hl-dark.css" },
]
```

With the new format:

```toml
# New
[markdown]
github_alerts = true  # optional, enables GitHub-style callout syntax

[markdown.highlighting]
style = "class"
light_theme = "github-light"  # or any zola built-in theme
dark_theme = "github-dark"    # or any zola built-in theme
```

You should also delete the generated `hl-light.css` and `hl-dark.css` files from your `static/` directory if you have them.
The new generated css files for code highlighting will be `giallo-light.css` and `giallo-dark.css`. (giallo is zola's new syntax highlighter)

## [5.5.0] - 2025-10-18

- feat: shortcode `figure`'s `caption` can use markdown now, the `via` prop is removed, use `caption="[via](https://example.com)"` instead
- fix: shortcode `figure` now will keep img's ratio on small screens when `width` and `height` is set
- ui: a few tweaks

## [5.4.3] - 2025-09-29

- fix: correct back link button behavior when there is a hash in url

## [5.4.2] - 2025-09-27

- fix: feed.xml template build error when there is no post

## [5.4.1] - 2025-09-21

- feat: add overflow scroll indicator for table-of-contents

## [5.4.0] - 2025-09-06

- ui: a few tweaks
- feat: back link button now do `history.back()` if appropriate
- feat: add custom height and width options for `figure` shortcode [@znxftw](https://github.com/znxftw) ([#83](https://github.com/isunjn/serene/pull/83))

## [5.3.1] - 2025-05-22

- fix: `extra_syntaxes_and_themes` defaults to theme's, so you don't need to copy that path [@makai410](https://github.com/makai410) ([#80](https://github.com/isunjn/serene/pull/80))

## [5.3.0] - 2025-05-07

- ui: a few tweaks

## [5.2.1] - 2025-04-19

- fix: homepage avatar style

## [5.2.0] - 2025-04-10

- ui: a few tweaks

## [5.1.0] - 2025-04-08

- feat: add support for zola v0.20.0 codeblock [name annotation](https://www.getzola.org/documentation/content/syntax-highlighting/#annotations), previous `codeblock` shortcode is deprecated
- refactor: remove `static/` prefix from icon paths, eliminates the need to copy icon files [@koyokr](https://github.com/koyokr) ([#76](https://github.com/isunjn/serene/pull/76))

## [5.0.1] - 2025-03-31

- fix: external links & recent posts on the homepage [@Hiramiya](https://github.com/Hiramiya) ([#74](https://github.com/isunjn/serene/pull/74))

## [5.0.0] - 2025-03-20

> **Warning**
>
> This version is a big redesign and contains lots of breaking changes.
> If you came from a previous version and want to upgrade, I suggest you start all over again.

- New style: headerless, section title and subtitle, improved typography...
- Collections: special blocks for showcasing your list (a more general form of previous 'projects' page)
- Default icon size changed from 20 to 18
- Some config options are moved from `config.toml` to specific `_index.md`
- Added options: `date_format` `back_link_text`, (section) `title` `subtitle`, (homepage) `footer`
- Removed options: `display_*` `nav_*` `blur_effect` `display_tags` `truncate_summary` `not_found_title`
- Callouts: `question` removed, `alert` renamed to `caution`, attribution `header` renamed to `title`
- Typst math rendering removed
- Added CSS variables: `--primary-decoration-color` `--text-decoration-color` `--highlight-mark-color` `--font-size` `--line-height`
- Removed CSS variables: `--homepage-font-size` `--homepage-line-height` `--paragraph-font-size` `--paragraph-line-height` `--aside-font-size`
- Lots of UI tweaks
- feat: Support subpath `base_url` [@b-d-e](https://github.com/b-d-e) ([#68](https://github.com/isunjn/serene/pull/68))
- fix: `force_theme` option [@teh-banana](https://github.com/teh-banana) ([#71](https://github.com/isunjn/serene/pull/71))
- fix: Add content-type header to reaction fetch [@sorokya](https://github.com/sorokya) ([#72](https://github.com/isunjn/serene/pull/72))

## [4.5.0] - 2024-11-03

### UI:

- A few tweaks

## [4.4.0] - 2024-11-02

### Add:

- New feature: Anonymous emoji reactions
- New options: `display_bio` `display_avatar` `recent`

### Remove:

- Removed options: `homepage_layout` (use `recent` instead), `recent_more`
- Removed css variable: `--icon-size`

### UI:

- A few tweaks

## [4.3.0] - 2024-10-13

### Add:

- Add katex [copy-tex](https://github.com/KaTeX/KaTeX/tree/main/contrib/copy-tex) extension & bump katex version to 0.16.11

## [4.2.0] - 2024-10-04

### Fix:

- Fix anchor link style issue, now `#` should no be present in the RSS file

## [4.1.0] - 2024-09-16

### Add:

- `force_theme` option to only use light or dark theme [@bruceoberg](https://github.com/bruceoberg) ([#62](https://github.com/isunjn/serene/issues/62))
- A few more icons [@bruceoberg](https://github.com/bruceoberg) ([#63](https://github.com/isunjn/serene/issues/63))


## [4.0.0] - 2024-08-11

- Deal with breaking changes of zola 19 config options:
>  - Changed config options named `generate_feed` to `generate_feeds` (both in config.toml and in section front-matter)
>  - Changed config option `feed_filename: String` to `feed_filenames: Vec<String>`

## [3.4.0] - 2024-04-25

### Add:

- Math rending with [Typst](https://typst.app) [@Lambdaris](https://github.com/Lambdaris) ([#57](https://github.com/isunjn/serene/pull/57))

## [3.3.1] - 2024-03-10

### Fix:

- Callout content overflow issue

### UI:

- Change highlight color of `diff` syntax
- A few tweaks

## [3.3.0] - 2024-03-01

### Add:

- New css variables: `--callout-border-radius` `--detail-border-radius`

### Fix:

- Overflow issue on mobile screens

### UI:

- Update quote icon
- A few tweaks

## [3.2.0] - 2024-01-26

### Add:

- Dark mode img/chart brightness option

## [3.1.0] - 2024-01-20

### Add:

- New shortcode: `quote` and `detail`

### Fix:

- Add `word-wrap: break-word` to inline code


## [3.0.0] - 2024-01-14

> **Warning**
>
> This version contains several breaking changes.
> If you came from a previous version and want to upgrade, I suggest you start all over again.

### Add:

- `recent` homepage layout
- `featured` mark
- Add title to ToC when it's too long
- A way to sort categories
- Project item image
- prerender/prefetch when hover, using `speculationrules` or `prefetch`
- RSS mask
- A few more css variables

### Fix:

- Theme init logic
- Mobile sidebar ui

### UI:

- A few tweaks
- Default icon size set to 20 (You should re-copy the `static/icon` folder)


## [2.3.0] - 2024-01-09

### Fix:

- `z-index` of mobile sidebar

### UI:

- Color change and some small tweaks

### Remove:

- Default custom font removed


## [2.2.1] - 2024-01-02

### Fix:

- Use `sessionStorage` for theme init in `_base.html`

## [2.2.0] - 2023-12-29

### Fix:

- Use `sessionStorage` for theme restore
- Fix an issue when initializing giscus theme
- Hide `#` anchor link in feed file

## [2.1.2] - 2023-09-19

### Fix:

- Outdate alert not 'hidden' ([#49](https://github.com/isunjn/serene/issues/49))

## [2.1.1] - 2023-09-16

### Add:

- Custom 404 page

## [2.0.1] - 2023-09-13

### Fix:

- Min height of prose page & post page

## [2.0.0] - 2023-09-01

> **Warning**
>
> This version contains several breaking changes.
> If you came from a previous version and want to upgrade, I suggest you start all over again.

### UI:

- Text selction now is styled
- Other minor tweaks
- Change defalut bg color of codeblock to transparent

### Add:

- Option `dispaly_tags` and `truncate_summary` [@woojiq](https://github.com/woojiq) ([#40](https://github.com/isunjn/serene/issues/40))
- Support for footnote and backlink
- Active TOC indicator
- Generay `prose` section/page
- Config option `sections`, now you can rename `blog` to somthing else, e.g. `posts`
- Support for header nav fold/unfold
- Option for homepage layout, can be `about` or `list`
- A separate `_custom_css.html` for css customization

### Fix:

- Codeblock distance calculation
- Codeblock highlight style
- Add description tag only when it's available
- Post 3 column layout issue
- Inline code style in list item
- Link text-decoration style on mobile


## [1.2.0] - 2023-08-19

### UI:

- Use noborder theme of giscus by default
- Post list item and callout styles changed
- Code block styles improved
- Default colors changed

### Add:

- Outline styles [@mrtnvgr](https://github.com/mrtnvgr) ([#26](https://github.com/isunjn/serene/pull/26))
- Support self-host font ([#29](https://github.com/isunjn/serene/pull/29))
- Copy button for code blocks ([#30](https://github.com/isunjn/serene/pull/30))
- Support light/dark switch for code blocks ([#33](https://github.com/isunjn/serene/pull/33))
- Support tags for project page
- Back-to-top button
- A shortcode for code block with file name: `codeblock` ([#39](https://github.com/isunjn/serene/pull/39))

### Fix:

- Update theme toggle icon on page load [@mrtnvgr](https://github.com/mrtnvgr) ([#25](https://github.com/isunjn/serene/pull/25))
- Layout shift problem on post page ([#27](https://github.com/isunjn/serene/pull/27))

## [1.1.1] - 2023-08-09

- Allow no tags in front matter [@mrtnvgr](https://github.com/mrtnvgr)
- Fix figcaption width issue

## [1.1.0] - 2023-05-27

- Fix theme auto-toggle logic
- A few ui tweaks

## [1.0.0] - 2023-05-24

> **Warning**
> The 1.0.0 version contains many breaking changes.
> If you came from a previous version and want to upgrade, I suggest you start all over again.

### Breaking

- `config.toml` restructured, config items are renamed
- All analytics configs removed, use `_head_extend.html` instead
- All comment-support configs removed, replace with [giscus](https://giscus.app)
- Icons now using svg files
- Callout renamed: `info -> note`, `caution -> warning`, `warning -> alert`
- Callout removed: `good`, `bad`, `happy`, `unhappy`, `check`, `wrong`, `flag`, `star`
- `cc_license` removed
- Reading-progress-bar removed
- Back-to-top button removed
- Many other tweaks

## [0.2.0] - 2022-02-16

### Add:
- KaTeX support
- Mermaid support

### Fix:
- Style issue of table-of-contents
- A few non-critical bugs

## [0.1.0] - 2022-01-14

First release 🎉

[6.0.0]: https://github.com/isunjn/serene/compare/v5.7.0...v6.0.0
[5.7.0]: https://github.com/isunjn/serene/compare/v5.6.3...v5.7.0
[5.6.3]: https://github.com/isunjn/serene/compare/v5.6.2...v5.6.3
[5.6.2]: https://github.com/isunjn/serene/compare/v5.6.1...v5.6.2
[5.6.1]: https://github.com/isunjn/serene/compare/v5.6.0...v5.6.1
[5.6.0]: https://github.com/isunjn/serene/compare/v5.5.0...v5.6.0
[5.5.0]: https://github.com/isunjn/serene/compare/v5.4.3...v5.5.0
[5.4.3]: https://github.com/isunjn/serene/compare/v5.4.2...v5.4.3
[5.4.2]: https://github.com/isunjn/serene/compare/v5.4.1...v5.4.2
[5.4.1]: https://github.com/isunjn/serene/compare/v5.4.0...v5.4.1
[5.4.0]: https://github.com/isunjn/serene/compare/v5.3.1...v5.4.0
[5.3.1]: https://github.com/isunjn/serene/compare/v5.3.0...v5.3.1
[5.3.0]: https://github.com/isunjn/serene/compare/v5.2.1...v5.3.0
[5.2.1]: https://github.com/isunjn/serene/compare/v5.2.0...v5.2.1
[5.2.0]: https://github.com/isunjn/serene/compare/v5.1.0...v5.2.0
[5.1.0]: https://github.com/isunjn/serene/compare/v5.0.1...v5.1.0
[5.0.1]: https://github.com/isunjn/serene/compare/v5.0.0...v5.0.1
[5.0.0]: https://github.com/isunjn/serene/compare/v4.5.0...v5.0.0
[4.5.0]: https://github.com/isunjn/serene/compare/v4.4.0...v4.5.0
[4.4.0]: https://github.com/isunjn/serene/compare/v4.3.0...v4.4.0
[4.3.0]: https://github.com/isunjn/serene/compare/v4.2.0...v4.3.0
[4.2.0]: https://github.com/isunjn/serene/compare/v4.1.0...v4.2.0
[4.1.0]: https://github.com/isunjn/serene/compare/v4.0.0...v4.1.0
[4.0.0]: https://github.com/isunjn/serene/compare/v3.4.0...v4.0.0
[3.4.0]: https://github.com/isunjn/serene/compare/v3.3.1...v3.4.0
[3.3.1]: https://github.com/isunjn/serene/compare/v3.3.0...v3.3.1
[3.3.0]: https://github.com/isunjn/serene/compare/v3.2.0...v3.3.0
[3.2.0]: https://github.com/isunjn/serene/compare/v3.1.0...v3.2.0
[3.1.0]: https://github.com/isunjn/serene/compare/v3.0.0...v3.1.0
[3.0.0]: https://github.com/isunjn/serene/compare/v2.3.0...v3.0.0
[2.3.0]: https://github.com/isunjn/serene/compare/v2.2.1...v2.3.0
[2.2.1]: https://github.com/isunjn/serene/compare/v2.2.0...v2.2.1
[2.2.0]: https://github.com/isunjn/serene/compare/v2.1.2...v2.2.0
[2.1.2]: https://github.com/isunjn/serene/compare/v2.1.1...v2.1.2
[2.1.1]: https://github.com/isunjn/serene/compare/v2.0.1...v2.1.1
[2.0.1]: https://github.com/isunjn/serene/compare/v2.0.0...v2.0.1
[2.0.0]: https://github.com/isunjn/serene/compare/v1.2.0...v2.0.0
[1.2.0]: https://github.com/isunjn/serene/compare/v1.1.1...v1.2.0
[1.1.1]: https://github.com/isunjn/serene/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/isunjn/serene/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/isunjn/serene/compare/v0.2.0...v1.0.0
[0.2.0]: https://github.com/isunjn/serene/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/isunjn/serene/releases/tag/v0.1.0
