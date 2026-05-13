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
- Deleted 18 WordPress RSS feed pages

## Site Structure
- `index.html` — homepage (one-page site with #home, #about, #services, #blog, #contact sections)
- `SLUG/index.html` — individual blog posts (17 posts)
- `category/SLUG/index.html` — category filtered views (18 categories)
- `YYYY/MM/index.html` — archive filtered views (15 month archives)
- `wp-content/` — theme CSS, images, uploads (keep these)
- `wp-includes/` — theme JS (keep these)

## Key Details
- Brand color: `#be1e2d`
- Contact form: https://formsubmit.co (action="https://formsubmit.co/info@boulderqa.com")
- Category/archive pages use injected JS to filter articles client-side (articles have CSS classes like `category-agile` and `datetime` attributes already embedded from WordPress)

## To-Do Before Going Live
1. Update DNS at GoDaddy — 4 A records pointing to GitHub Pages IPs + CNAME www → cwinski-r36.github.io
2. Add custom domain (www.boulderqa.com) in GitHub Pages settings
3. Verify SSL certificate is active after DNS propagates
4. Submit sitemap to Google Search Console (https://www.boulderqa.com/sitemap.xml)
5. Test contact form — first submission triggers a Formsubmit.co verification email to info@boulderqa.com
6. Cancel GoDaddy WordPress hosting plan (do this last)
