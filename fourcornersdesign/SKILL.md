---
name: "fourcornersdesign"
description: "Redesign pages, headers and templates on WordPress sites built with the BBZ Fourcorners theme (e.g. Smile Design Boutique, Excellence Medical Boutique) through Claude in Chrome."
---

# Fourcorners Design (BBZ Fourcorners WordPress theme)

Use this when the user asks to design, redesign or restyle pages, sections, headers, footers, posts or templates on a site running the **bbz-fourcorners** theme, working in the user's logged-in Chrome via Claude in Chrome.

## Ground rules (always)
- Use the site's own theme colours only, via CSS vars / block colour presets. Never invent colours. Check each site's palette first (read `--color-*` vars or `getSettings().colors`). Known: gold `--color-primary` #b08d54, navy `--color-heading` #09053a, `--color-text`, dirty white `--color-site-background` #fcfbf8, `--color-light-grey`, white; Excellence also has `offwhite`, `purple`, `purple-50`, `white-20`, `white-50`, `dark-transparent`. Fonts `--font-family-heading` (Bodoni Moda) / `--font-family-text` (Montserrat); block font-family slugs `text-font`, `heading-font`, `decorative-font`. Font-size presets: small, medium, large, h6–h1, display, decorative. Radius presets xs/sm/md/lg (values differ per site, e.g. Excellence 8/12/26/32px).
- Block settings first: use native block settings (colour, typography, spacing, border, radius, aspect ratio, layout/grid, block gap, sticky position, theme block options) before writing CSS. Only use CSS for what settings cannot do (hover/transitions, pseudo-elements, :has(), theme ACF block internals, breakpoint-specific fixes, backdrop-filter, opacity colours — see gotcha below). Reusable elements should be synced patterns so settings stay consistent.
- Each site has its OWN style. Do not copy one site's look onto another unless asked.
- Never edit theme `build/` files or ACF JSON. Theme PHP is read-only (read it via `theme-editor.php?file=` fetch). No plugin installs, settings changes or permanent deletes without explicit permission.
- Do not touch other patterns/templates/pages than the ones requested. When a shared element (e.g. Primary menu, global header) would change other pages, ask first or build a separate copy.
- Reference / framework sites (e.g. fourcornersgutenberg.kinsta.cloud) are READ-ONLY: inspect via GET/REST reads only; never save, update, publish or change settings there. If unsure whether an action writes, stop and ask.
- Redesign/preview work must be isolated: create NEW pages/patterns (clear names like "X V2 Design", draft/private where suitable), never overwrite originals or global header/footer. For page-specific header/footer, insert a duplicated header/footer pattern in the page and hide the global one only for that page ID; prefer page-scoped CSS so global settings stay unchanged when isolation is required.
- Preserve existing copy, especially SEO pages: never delete, shorten or rewrite SEO text; restyle and reorganise only (accordions/tabs for long content), and verify all original text is still present afterwards.
- Make every change revertible: before editing CSS, save a full copy of Additional CSS (e.g. private page "Backup – Additional CSS before X (date)" with a `core/code` block, then verify the round-trip text matches exactly). Note the latest revision ID of every post you edit; restore via `/wp/v2/{type}/{id}/revisions/{rev}` content if needed.
- CSS rules: no comments, mobile-first, scoped class names, added to Customizer Additional CSS.
- Placeholder/dummy content must be clearly labelled (e.g. before/after placeholder note). Never present fabricated results or quotes as real; flag invented quotes, stats and medical copy for client review.
- Preferred visual direction (Smile site, current): light sections alternating `white` / `background`, gold accents, navy footer only (dark blue allowed for accent cards e.g. Where-to-find-us left column); buttons 4px radius; content width matches header (1360px max) on all pages; gold mobile burger + gold Book button in mobile header.
- Excellence site keeps its own style: navy hero/sections with purple glow, offwhite, floating dark header.
- When asked for an estimate or explanation, answer directly with clear assumptions; do not start building.

## How the theme works
- Classic theme. `singular.php` = `get_header(); the_content(); get_footer();` — every single page/post/CPT renders ONLY its own block content. No auto title/featured image. Layout lives in each post's blocks; "templates" are synced patterns (`wp_block`) copied in manually.
- Global header/footer = synced patterns chosen by hidden options `_bbz_global_header_block_id` / `_bbz_global_footer_block_id` (edit at `wp-admin/options.php`, set the input, click Save Changes — note the form has an input named `action`, so `form.action` is shadowed; click the real button).
- Header gets `.is-on-media` only when the page starts with a theme hero block; on custom dark heroes force the white logo with `body:has(.your-dark-hero) .site-header .wp-block-site-logo.has-white-logo .custom-logo{opacity:0}` + `.site-logo-white{opacity:1}`. Header gets `.is-hidden` on scroll-down.
- Key blocks: `acf/hero` (bg image/video, `overlayGradient`, class `hero-treatment`), `acf/hero-split` (inner `.block-hero-split__content` group), `acf/treatment-grid` (data: source all/latest/specific, post_type treatment/post/page, `perRow`, styles regular/slider/slider-text; no pagination), `acf/employee-list` (styles default grid / `is-style-preview` slider where per-view = count+1), `acf/before-after` (`data.beforeafter` id), `acf/reviews`, `acf/content-tabs`+`acf/content-tab` (FAQ tabs with `core/accordion`), `acf/form` (data `block_form_gravity_form` / `field_form_gravity_form`, `block_form_layout` stacked|horizontal), `acf/menu-navigation`, `acf/icon-list`, `core/icon` (icons like `bbz/star`, `bbz/tooth`, `bbz/tooth-shield`, `bbz/tooth-sparkle`, `bbz/hourglass`, `bbz/hand-heart`, `bbz/dental-mirror`, `bbz/clock`, `bbz/calendar-tooth`).
- Wide width var: `--wp--style--global--wide-size`; sections use `alignwide` columns.
- GOTCHA: a colour preset named "text" generates `.has-text-color{color:var(--text) !important}` in global styles, which overrides ANY custom (non-preset) text colour set in block settings (e.g. rgba white 90%). Use preset colours in settings; keep custom/opacity colours as one-line CSS.
- `acf/form` button/field colours are block attributes (`buttonBackgroundColor`, `buttonTextColor`, etc.); `acf/treatment-grid` columns via `perRow`; `acf/employee-list` via `columns`/`count`; `acf/content-tabs` active tab colour via `tabActiveBackgroundColor` — use these instead of CSS.
- `acf/content-tabs` + white card groups also make a CSS-free reviews/testimonials carousel (tabs 01–04).

## Working technique (Claude in Chrome)
1. Open a block-editor page (`post.php?post=ID&action=edit`, wait ~4s) to get `wp.blocks`, `wp.data`, `wp.apiFetch`. Front-end pages have no `wp.apiFetch`.
2. Inspect structure: walk `wp.data.select('core/block-editor').getBlocks()` printing name/className/content (strip URLs and `?&=` from output or the tool blocks it). Keep each JS call small; long loops/timeouts fail in the extension — split into steps.
3. Edit:
   - In-editor: `updateBlockAttributes`, `replaceInnerBlocks`, `replaceBlock`, `moveBlockToPosition`, then `wp.data.dispatch('core/editor').savePost()`. Never pass stale block objects to `resetBlocks` after updating them (reverts edits).
   - Via REST: parse raw content with `wp.blocks.parse`, clone/modify with `cloneBlock`/`createBlock`, `serialize`, POST to `/wp/v2/{pages|posts|treatment|employee|beforeafter|blocks}/ID`. Good for bulk rollouts across many posts.
   - Reuse sections by cloning from a reference page (e.g. parse the main treatment layout and the homepage "Where to find us").
   - Hand-built HTML (mega menu, mobile drawer) goes in `core/html` inside a synced pattern; CSS-only toggles via hidden checkbox + `:has()`, submenus via `<details>`.
4. CSS: in `wp-admin/customize.php` (wait ~2.5s): `const s=wp.customize('custom_css[bbz-fourcorners]'); s.set(s.get()+css); wp.customize.previewer.save()`. To modify existing rules, split/join exact strings and report whether the match was found.
5. Lighten dark sections: set `backgroundColor` to `white`/`background`, delete `style.background`, swap text colours `background|white` -> `heading` (headings) / `text` (paragraphs) recursively, keep gold fill buttons, swap `white-20` borders to `primary`, `reviews-gold` -> `reviews-light`.
6. Media: search the library first; for new images fetch from Pexels (`images.pexels.com/photos/ID/pexels-photo-ID.jpeg?w=1200`) -> blob -> FormData POST `/wp/v2/media` with alt text; view the image before using it.
7. Gravity Forms REST may be off: create forms via Forms > Import/Export > Import Forms by POSTing a JSON Blob with the page's nonce fields.
8. If CPT singles 404 at pretty URLs, re-save Settings > Permalinks (no changes) to flush rules.
9. Before bulk-editing via REST, check `parse`+`serialize` round-trips unchanged (compare whitespace-stripped raw vs re-serialized) so other blocks are not altered.
10. Match classes by token (`className.split(/\s+/).includes('x')`), never `\b` regex — `treatment-glance__label` would also match `treatment-glance__label-row`.
11. Before removing any CSS rule, search ALL content (pages, posts, treatment, employee, blocks; all statuses) for the class and apply the same block settings everywhere it is used.
12. The extension fails on long/heavy scripts: split work, store helper source and snapshots in `localStorage` (survives navigation on the same origin), and avoid output containing URLs/query strings (it gets blocked).

## Verification (always before reporting)
- Desktop: inject an iframe of the page at 1500–1700px wide, scaled (`transform:scale(.37–.6)`), scroll it, screenshot/zoom. Measure alignment with `getBoundingClientRect` (header bar vs `.alignwide`).
- Mobile: iframe at 390px (and 360px); toggle drawers/details via JS before screenshotting.
- Check hover/visibility issues (white text on light bg, invisible outline buttons, logo contrast) and fix before replying.
- CSS-to-settings refactors must be proven identical: snapshot computed styles (colour, font size/family, line height, letter spacing, transform, margins, width/height, aspect ratio, object-fit, radius, borders, rects) of affected elements before changes (desktop directly in the tab, mobile in a 390px iframe); after applying settings, delete the target rules from the live CSSOM (`#wp-custom-css` sheet) and diff against the baseline; only remove rules from Additional CSS when diffs are zero, then re-diff on the live site. Compare desktop at the same real viewport (iframes add scrollbar width and change fluid sizes).
- For isolated/redesign work also confirm original pages' content and modified dates are unchanged and reference sites were never written to.

## Proven layouts to reuse
- Treatment page: light hero + white "at a glance" card (`treatment-glance glance-light`), feature strip, About + Real Results (before/after), Meet the Dentists cards, How it works zigzag (square images, centred title), centred FAQ with tabs, Patient stories (`reviews-light`), Where to find us (dark navy left card + map, gold Get Directions), Book CTA last.
- Concern pages: same as treatment + "How we can help" recommended treatments.
- Dentist profile: light hero with photo masked into background + specialties list, feature strip, about + video + navy quote panel, treatments grid, qualifications, before/after sliders.
- Blog single: hero (breadcrumb, gold category/date, title with gold italic, excerpt, meta, rounded image), article column + sticky sidebar (navy consult card, related links), related `core/query` cards, Book CTA. Blog listing: `core/query` with pagination, 3-col cards.
- Before & After case: dark hero, before/after pairs with in-image dirty-white labels, treatment-focus panel, related transformation sliders; lives in the `beforeafter` CPT.

## Reporting
Short summary of what changed per section, what was verified, anything invented/placeholder, how to revert (backup page ID, revision IDs), and the next sensible step as one question. When asked what custom CSS exists, group rules by section/class with counts and say which are global, page-specific or unused.