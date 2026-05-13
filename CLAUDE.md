# Boulder QA Website

## Project Goal
Convert www.boulderqa.com from a WordPress site (hosted on GoDaddy at $300+/year) to a free static site hosted on GitHub Pages. No content changes — same site, zero hosting cost.

## Hosting
- **GitHub repo**: https://github.com/cwinski-r36/boulderqa-website
- **GitHub Pages branch**: `main`, root `/`
- **Target domain**: www.boulderqa.com
- **Local dev**: `python3 -m http.server 8080` from `boulderqa.com/`

## What Was Done
- Crawled the live WordPress site with `wget --mirror`
- Replaced the WordPress contact form with Formsubmit.co (sends to info@boulderqa.com)
- Removed all WordPress/GoDaddy junk: wp-admin, wp-json, mu-plugins, RevSlider, forms-contact plugin, flexslider, RSS feed pages
- Fixed all internal blog links (WordPress `?p=ID` URLs → slug URLs)
- Fixed nav links on all pages (blog posts, category pages, archive pages) to correctly point to homepage sections (#home, #about, etc.)
- Added SEO: meta description, Open Graph tags, LocalBusiness JSON-LD schema, canonical URL
- Added favicon (ICO + PNG sizes)
- Created sitemap.xml and robots.txt
- Removed broken WordPress `<link>` tags (wp-json, oEmbed, feed, wp-login)
- Removed Meta sidebar widget and Recent Comments widget from all pages
- Removed search widget from all pages (doesn't work on static sites)
- Added JS filtering to category and archive pages so they show only matching posts
- Removed all GoDaddy CSS references
- Deleted 18 WordPress RSS feed pages (`*/feed/index.html`)
- Added logo link wrapper on all pages so logo click goes to homepage
- Added dynamic copyright year to all pages using `document.write(new Date().getFullYear())`
- Fixed duplicate post on agile category page (removed secondary `for_current_wp_query` grid from all 18 category pages)
- Created branded 404 page (`404.html`) — GitHub Pages serves this automatically for missing URLs
- Stripped broken URL-encoded WP asset `<link>`/`<script>` tags from all 51 pages — wget URL-encoded `?ver=8.44` → `%3Fver=8.44` in file paths, making them unloadable; removed ~12-13 dead tags per page (**NOTE: this also incorrectly removed the Impreza CSS and JS — see Site Structure for the full fix history**)
- Renamed all wget-downloaded assets that had `?ver=X.X` in their filenames to clean names — 7 JS files, 6 woff2 font files, 1 CSS file; updated all HTML references accordingly
- Restored the Impreza main CSS stylesheet (`wp-content/themes/Impreza/css/style.min.css`, 495KB) which was stripped and never restored — its absence caused a complete layout regression (no grid, no nav styling)
- Added missing post title `<h2>` headings to 11 of 17 blog posts — the other 6 already had a heading in their content; titles extracted from each page's `<title>` tag
- Fixed one content link in blunders post (`index.html%3Fus_main_page_section=contact.html` → `index.html#contact`)
- Ran full site audit: all nav links, logo links, copyright, JS filters, contact form, viewport meta, and search widget removal confirmed clean across all 52 pages
- Moved `boulder-qas-2023-recommended-browser-device-list.html` (root-level file) into proper directory structure — it was the only blog post not in a `SLUG/index.html` directory, causing its sitemap URL to 404
- Fixed 35 archive/category pages that had absolute `https://boulderqa.com/YEAR/MONTH/...` empty anchor links pointing to the old WordPress site (replaced with correct relative paths)
- Added descriptive alt text to 14 client logo images on the homepage
- Live preview URL: https://cwinski-r36.github.io/boulderqa-website/ (subdirectory hosted, JS filters and relative paths all work correctly here)

## Site Structure
- `index.html` — homepage (one-page site with #home, #about, #services, #blog, #contact sections)
- `SLUG/index.html` — individual blog posts (17 posts)
- `category/SLUG/index.html` — category filtered views (18 categories)
- `YYYY/MM/index.html` — archive filtered views (15 month archives)
- `404.html` — custom branded 404 page
- `wp-content/` — theme CSS, images, uploads (keep these)
- `wp-includes/` — theme JS (keep these)
- **CRITICAL — The following Impreza/jQuery JS files MUST be referenced in every page's `<script>` tags** (with correct relative path prefix per depth). Removing them breaks the entire page layout. This mistake was made once and caused a complete visual regression.
  - `wp-includes/js/jquery/jquery.min.js`
  - `wp-content/themes/Impreza/common/js/base/passive-events.min.js`
  - `wp-content/themes/Impreza/common/js/vendor/magnific-popup.js`
  - `wp-content/themes/Impreza/common/js/base/header.no-cache.min.js`
  - `wp-content/themes/Impreza/js/us.core.min.js`
  - `wp-content/themes/Impreza/common/js/vendor/owl.carousel.js`
  - `wp-content/themes/Impreza/common/js/vendor/royalslider.js`
- These files were originally downloaded by wget with `?ver=X.X` in their filenames. Chrome normalizes `%3F` → `?` before HTTP requests, causing Python's http.server to strip the query string and fail to find the file. All files were renamed to clean names and all `<script src="...%3Fver=...">` tags were updated to use clean paths.
- Font files (`wp-content/themes/Impreza/fonts/fa-*.woff2`, `material-icons.woff2`) were similarly renamed from `?ver=8.44` names and their `@font-face` url() references updated.
- **CRITICAL — The Impreza CSS file MUST also be referenced in every page's `<head>`** as a `<link rel='stylesheet' id='us-main-css' href='[PREFIX]wp-content/themes/Impreza/css/style.min.css' media='all' />`. Without it the entire site renders as unstyled text — no nav, no grid, no layout. It was originally stripped during the %3Fver cleanup and caused a full visual regression. PATH PREFIX: none at depth 0 (index.html), `../` at depth 1 (slug/), `../../` at depth 2 (category/slug/ or YYYY/MM/).

## Key Details
- Brand color: `#be1e2d`
- Google Fonts: Dosis (headings/UI) + PT Sans (body)
- Contact form: https://formsubmit.co (action="https://formsubmit.co/info@boulderqa.com")
- Category/archive pages use injected JS to filter articles client-side (articles have CSS classes like `category-agile` and `datetime` attributes already embedded from WordPress)
- Root-relative favicon paths (`/favicon.ico` etc.) work on production domain but not localhost — this is expected

## JS Filter Snippets

**Category pages** (injected before `</body>`):
```javascript
(function(){
  var path = window.location.pathname.replace(/\/index\.html$/, '/').replace(/\/+$/, '');
  var parts = path.split('/');
  var catIdx = parts.indexOf('category');
  if (catIdx === -1) return;
  var slug = parts[catIdx + 1];
  if (!slug) return;
  var cls = 'category-' + slug;
  document.querySelectorAll('article.w-grid-item').forEach(function(art){
    if (!art.classList.contains(cls)) art.style.display = 'none';
  });
})();
```

**Archive pages** (injected before `</body>`):
```javascript
(function(){
  var path = window.location.pathname.replace(/\/index\.html$/, '/').replace(/\/+$/, '');
  var parts = path.split('/').filter(Boolean);
  var mm = parts[parts.length - 1];
  var yyyy = parts[parts.length - 2];
  if (!yyyy || !mm || !/^\d{4}$/.test(yyyy) || !/^\d{2}$/.test(mm)) return;
  var prefix = yyyy + '-' + mm;
  document.querySelectorAll('article.w-grid-item').forEach(function(art){
    var time = art.querySelector('time[datetime]');
    if (!time || time.getAttribute('datetime').indexOf(prefix) !== 0) {
      art.style.display = 'none';
    }
  });
})();
```

## Python Bulk-Edit Scripts
- `inject_filters.py` — injects category/archive JS into all matching pages (idempotent)
- `inject_css_link.py` — injects the Impreza CSS `<link>` tag into all pages at correct depth (idempotent, checks for `Impreza/css/style.min.css` before injecting)
- `inject_post_titles.py` — injects missing `<h2>` post title into blog posts whose content starts with `<p>` (skips pages that already have a heading)
- `fix_ver_names.py` — renames `?ver=X.X` files to clean names and updates all HTML references (historical, already run)
- Other one-off scripts were run inline for nav link fixes, GoDaddy removal, search widget removal

## To-Do Before Going Live
1. Update DNS at GoDaddy — 4 A records pointing to GitHub Pages IPs + CNAME www → cwinski-r36.github.io
2. Add custom domain (www.boulderqa.com) in GitHub Pages settings
3. Verify SSL certificate is active after DNS propagates
4. Submit sitemap to Google Search Console (https://www.boulderqa.com/sitemap.xml)
5. Test contact form — first submission triggers a Formsubmit.co verification email to info@boulderqa.com
6. Cancel GoDaddy WordPress hosting plan (do this last)

## Verified Working on Live GitHub Pages Preview
- JS category/archive filters work at the `/boulderqa-website/` subdirectory path — the `indexOf('category')` approach finds the slug correctly regardless of path prefix
- Font Awesome icon fonts load correctly (files renamed to clean names, `@font-face` url() references updated)
- All relative paths resolve correctly for all page depths (root, 1-level, 2-level slugs)
- 404.html is served by GitHub Pages for missing URLs
- No PHP files, SQL dumps, wp-config, `.env`, or credentials anywhere in the repo

## Known Non-Issues (False Positives in Link Audits)
- Anchor links (`../../index.html#home`, etc.) — audit scripts treat `#anchor` as a file path; these work fine in browser
- Root-relative favicon paths (`/favicon.ico`, `/favicon-32x32.png`, `/favicon-192x192.png`, `/apple-touch-icon.png`) — break on localhost but work correctly on production domain
- `widget_search` string in theme CSS (`<style id="us-theme-options-css">`) — it's a CSS class selector, not an actual search widget element; the search widget HTML was fully removed
- Logo audit: the logo `<img>` is wrapped in `<a>` but has a `<div class="w-image-h">` between them — naive regex checks will miss it; the links are correct

## Impreza Theme
The Impreza WordPress theme is frozen at v8.44. Theme updates are irrelevant — there is no WordPress installation, no update mechanism, and no server-side code. The theme's assets are just static files.
