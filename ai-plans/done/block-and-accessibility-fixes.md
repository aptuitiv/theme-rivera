# Plan: Rivera block and accessibility fixes

Created: 2026-09-12

Found while comparing the Skeleton theme with Carmine and Harvest. Carmine is the reference for block templates unless an item says otherwise.

**Useful commands**

```bash
# Compare a Rivera file with Carmine (run from the Themes folder)
diff -w -B Rivera/src/<path> Carmine/src/<path>

# Show a Carmine commit referenced below
git -C Carmine show <hash>
```

## Theme-specific fixes

- [x] **`full-width.twig`** — `<main>` has no `id="main"`, so the skip link has no target on this layout. Add it. Every layout (`one-column`, `narrow`, `full-width`, both two-column layouts) now has exactly one `<main id="main">`; all other templates extend one of these.
- [x] **Skip link CSS** — the skip link's `z-index: 100` can put it at or below the header's stacking level. Use Harvest's approach: `z-index: calc(var(--Header-sticky-z-index, 100) + 1)` (Harvest `src/css/components/header/skip-to-main.css`). Applied to Rivera's `.Header-skipToMain` in `css/components/header/header.css`.
- [x] **`blocks/image-grid.twig`** — `{% set width = 800 %}` and the `width:` values in `count2`–`count6` overwrite the block's Width field, so the Width setting never takes effect. Remove them (Carmine commit `1745710`).
- [x] **`blocks/image-row.twig`** — `href="{image.url}"` uses single braces, so the link is broken. Change to `href="{{ image.url }}"`, and drop `target="_blank"` or add a visually hidden "(opens in a new window)". Kept `target="_blank"`, added `rel="noopener"` and the visually hidden text.
- [x] **Margin/width support** — add `macros.blockMargin(margin)`/`macros.blockWidth(width, true)` to grid-2…6-columns, heading, html-code, and columned-content (Carmine commits `265e272`, `81c6c37`, `660fa93`). Confirm the CMS block definitions have Margin and Width fields. Grid files and columned-content also import `macros/macros` (Carmine's are missing it; without it the margin/width classes don't render). Rivera's columned-content already uses `ncClass`, not `nc_class`. The CMS block definitions can't be checked from this repo — confirm in the CMS.
- [x] **`blocks/google-ratings-bar.twig`** — each star is its own labelled image, so screen readers repeat "Star rating". The visible text already states the rating, so wrap the stars and number in `aria-hidden="true"` and use `iconAriaHidden` instead of `iconImg` (see Skeleton's `blocks/google-ratings-bar.twig`).
- [x] **Reviews link setting** — there are two "URL to view reviews" fields. Use the Settings one and remove the Styles one (Skeleton has this change):
    - [x] `blocks/google-ratings-bar.twig` — change `_core.theme.settings.googleRatingsBarReviewsLink` to `_core.theme.settings.customerRatingsBarReviewsLink` (3 places)
    - [x] `config/theme-styles.json` — in the "Blocks - Google Ratings Bar" group, remove the `googleRatingsBarReviewsLink` field and its "Review link" subgroup (the first one, which holds only that field). Keep the second "Review link" subgroup (the link typography).
    - Existing sites that set the link under Styles will need it re-entered under Settings → Customer Reviews & Ratings.
- [x] **`js/navigation/accessibility.js:270`** — `getParent()` stops climbing when `node.parentNode` has the `js-mainNav` class, but `navigation/main.twig` doesn't output `js-mainNav` on the menu `<ul>`. For top-level items the loop runs up to `<body>` and returns the wrong link, which breaks keyboard navigation between menu levels. Either add `js-mainNav` to the `<ul>` or change the check to `MainNav`. Added `js-mainNav` to the `<ul>` (matches Skeleton). Nothing else in Rivera selects `.js-mainNav` — `small-screen.js` uses `.js-navBar`.

## Accessibility fixes shared by all themes

Carmine isn't a good reference for these — each needs a new fix. Items marked *(verify)* were found in Skeleton, Carmine, and Harvest but haven't been checked in Rivera yet. (Rivera doesn't have the `rel="noopenner"` typo the other themes have.)

- [x] **Accordion isn't keyboard-operable** — the heading is `<div class="Accordion-heading js-accordionHeading">` with only a click listener. Use a `<button>` with `aria-expanded` and `aria-controls` (`blocks/accordion.twig`, `js/accordion.js`). *High* Also `css/components/accordion/accordion.css`: `width: 100%` on the heading and a transitioned `visibility: hidden` on closed content.
- [x] **Modals** — the close button `<button class="Modal-close" data-micromodal-close></button>` has no accessible name. Also check for a missing `aria-labelledby`, `disableFocus: true` on the popup, and the notification icon's missing `aria-hidden` *(verify)* (`widgets/collections/popups.twig`, `notifications.twig`). All confirmed and fixed as in Skeleton, plus `disableFocus` removed from `MicroModal.init()` in `js/main.js`. Kept Rivera's `Modal-slide` class.
- [x] **Pagination** — wrap in `<nav aria-label="Pagination">`, add `aria-current="page"` to the current page, and change the chevron icons from `role="img"` to `aria-hidden="true"` (`snippets/pagination.twig`).
- [x] **`iconImg` macro** outputs `<svg role="img" alt="…">` — `alt` isn't valid on `<svg>`. Use `aria-label` or a `<title>` (`macros/macros.twig`). Removed `alt`; the existing `<title>` + `aria-labelledby` name it.
- [x] **Form errors** — the form error container has no `role="alert"`/`aria-live`. Also check that `form.js` sets `aria-invalid`/`aria-describedby` *(verify)* (`macros/form-macros.twig`, `js/form.js`). It didn't; `form.js` now sets them from Just-Validate's `onValidate` and adds `role="alert"` to the error container.
- [x] **No `prefers-reduced-motion` CSS** — transitions, slider autoplay, and modal animations ignore it. Added to `css/base/base.css`.
- [x] **Mobile submenus hidden from screen readers** — submenus render with `aria-hidden="true"` and a tap doesn't change it; `aria-expanded` sits on the `<ul>` instead of the toggle *(verify)* (`navigation/main.twig`, `js/navigation/small-screen.js`). *High*
- [x] **Mobile menu** — no Escape to close, no focus trap, no focus return *(verify)*. Escape now closes the menu and returns focus to the menu button. No focus trap (not needed for the disclosure-style menu, same as Skeleton).
    - [ ] Rewriting the `role="menubar"`/`menuitem` menu as a disclosure pattern — Skipped — same as Skeleton.
- [x] **Main `<nav>`** — add `aria-label="Main"` *(verify)*. Replaced the redundant `role="navigation"` on `.NavBar` with `aria-label="Main"`. The menu button keeps its `aria-label="Navigation menu"` because it has no visible text.
- [x] **`title` as the only label** on social and logo links; social links open in a new window with no warning *(verify)*. Not applicable — the header logo link is named by the logo's alt text (falls back to the company name), header social icons are named by `iconImg`'s `<title>` and don't open a new window, and the footer has no social links. Added `rel="noopener"` and a visually hidden "(opens in a new window)" to the footer credit links (`snippets/footer.twig`).
- [x] **Required marker** — add `aria-hidden="true"` to the `*` in labels *(verify)*.
- [x] **Upload previews** use `alt="Image"` *(verify)* (`macros/form-macros.twig`). Now `alt="Preview of the uploaded image"`.
- [ ] **Video/audio** — no `<track>` captions or transcript option *(verify)*. Skipped — same as Skeleton.
- [x] **Landmark labels** — footer navs and sidebar `<aside>` elements have no `aria-label` *(verify)*. Footer `<ul>` wrapped in `<nav aria-label="Footer">` in `navigation/footer.twig`; sidebars are `<aside aria-label="Section navigation">` / `<aside aria-label="Sidebar">`.
- [ ] **`lang="en"` is hardcoded** in `snippets/header.twig` *(verify)*. Skipped — same as Skeleton.
- [ ] **Slider pause/play control** — Skipped — same as Skeleton.

## Verification

- [ ] `npm run build` completes and `npm run stylelint` shows no new warnings (stylelint: no new warnings; build not run)
- [ ] Every changed block renders in the CMS, including the Margin and Width options
- [ ] Keyboard check: the skip link works on the full-width layout; accordion headings open with Enter/Space

## Follow-up fixes (2026-09-13)

- [x] **`blocks/columned-content.twig` width override** — removed the `width:` values from `count2`–`count6` and every `{% set width = countN.width %}` line. They overwrote the block's Width field, so the Width setting never took effect (same fix as image-grid, Carmine commit `1745710`).
- [x] **`snippets/header.twig`** — the notifications widget was output twice (duplicate `id="notification-modal"`); it's now output once. The skip link and the SVG icon sprite moved out of `<header>` so the skip link is the first element in `<body>` (same order as Carmine and Skeleton). Removed the stray `</body>` at the end of the file (the footer closes the body).
- [x] **`blog/post.twig`** — the comment author's website link opens in a new window; added `rel="noopener"` and a visually hidden "(opens in a new window)" (same as Carmine).
