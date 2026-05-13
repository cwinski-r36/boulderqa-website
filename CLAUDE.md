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
- Stripped broken URL-encoded WP asset `<link>`/`<script>` tags from all 51 pages — wget URL-encoded `?ver=8.44` → `%3Fver=8.44` in file paths, making them unloadable; removed ~12-13 dead tags per page
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
- **CRITICAL — DO NOT REMOVE `%3Fver=` script/link tags for Impreza/jQuery files.** Both Python's `http.server` and GitHub Pages URL-decode `%3F` → `?` before looking up files, so they successfully serve files literally named `us.core.min.js?ver=8.44` on disk. Removing these tags breaks the entire page layout (Impreza's grid/nav/layout engine stops running). This mistake was made once — it caused a complete visual regression.
- The following JS files exist on disk with `?ver=` in the name and MUST remain referenced in every page's `<script>` tags (with correct relative path prefix per depth):
  - `wp-includes/js/jquery/jquery.min.js?ver=3.7.1`
  - `wp-content/themes/Impreza/common/js/base/passive-events.min.js?ver=8.44`
  - `wp-content/themes/Impreza/common/js/vendor/magnific-popup.js?ver=8.44`
  - `wp-content/themes/Impreza/common/js/base/header.no-cache.min.js?ver=8.44`
  - `wp-content/themes/Impreza/js/us.core.min.js?ver=8.44`
  - `wp-content/themes/Impreza/common/js/vendor/owl.carousel.js?ver=8.44`
  - `wp-content/themes/Impreza/common/js/vendor/royalslider.js?ver=8.44`
- Font files (`wp-content/themes/Impreza/fonts/fa-*.woff2?ver=8.44`) are referenced in inline CSS `@font-face` via `%3Fver=8.44` URLs — same mechanism, also confirmed working. Do not rename these files.
- The deleted plugin files (forms-contact, revslider) had `%3Fver=` tags correctly removed because those directories no longer exist on disk. The rule is: **if the file exists on disk, the `%3Fver=` script tag must stay.**

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
- Font Awesome icon fonts load correctly — GitHub Pages serves the `?ver=8.44`-named files when requested via `%3Fver=8.44` URL encoding
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
