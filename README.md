# The Slum Surcharge

The Daily Star, by Shamima Rita. Goes live at **https://campaign.thedailystar.net/syndicates-web/** on 3 October 2026.

This folder is the finished website. It's plain static files: no build step, no server code, no database and no calls to outside services. Every path inside it is relative, so it works both at the root of a domain (the Vercel preview) and inside the `/syndicates-web/` folder on the live server.

Don't edit these files by hand. They're generated from the project source (`site/`, command `npm run release`), which checks the text, the numbers, the calculator, accessibility and privacy before it writes this folder.

---


**Share cards on the preview:** link previews (Facebook, X, WhatsApp, Slack) point at the live address by design, so they show the photo only once the page is live at https://campaign.thedailystar.net/syndicates-web/.

---

## 2. Put it live at https://campaign.thedailystar.net/syndicates-web/

1. Copy **everything in this folder** into a folder named `syndicates-web` at the web root of `campaign.thedailystar.net`, so that `index.html` is served at https://campaign.thedailystar.net/syndicates-web/.
   - `vercel.json` and this `README.md` aren't needed on the live server. They're harmless if copied.
2. **Keep the trailing slash.** `https://campaign.thedailystar.net/syndicates-web` (no slash) must redirect to `https://campaign.thedailystar.net/syndicates-web/`. nginx and Apache do this for real folders by default. Without the slash, the relative paths break.
3. If the site sits behind Cloudflare, purge the cache for `/syndicates-web/` after each upload.

### Suggested response headers (optional, recommended)

| Path | Cache-Control |
|---|---|
| `index.html` | `no-cache` |
| `assets/*` (file names carry a content hash) | `public, max-age=31536000, immutable` |
| `media/*`, `brand/*` | `public, max-age=86400` |

Make sure these types are served (older servers may not know the first two): `.avif` → `image/avif`, `.webp` → `image/webp`, `.woff2` → `font/woff2`. On nginx, add any missing ones to `mime.types`. Don't use a `types {}` block inside a `location`: it replaces the whole list.

**nginx:**

```nginx
location /syndicates-web/ {
    index index.html;
    add_header Cache-Control "no-cache";
}
location /syndicates-web/assets/ {
    add_header Cache-Control "public, max-age=31536000, immutable";
}
location ~ ^/syndicates-web/(media|brand)/ {
    add_header Cache-Control "public, max-age=86400";
}
```

**Apache** (a `.htaccess` inside `syndicates-web/`, if `AllowOverride` permits it):

```apache
DirectoryIndex index.html
AddType image/avif .avif
AddType image/webp .webp
AddType font/woff2 .woff2
<IfModule mod_headers.c>
  Header set X-Content-Type-Options "nosniff"
  <FilesMatch "\.html$">
    Header set Cache-Control "no-cache"
  </FilesMatch>
  <If "%{REQUEST_URI} =~ m#/syndicates-web/assets/#">
    Header set Cache-Control "public, max-age=31536000, immutable"
  </If>
</IfModule>
```

---

## 3. Search engines and share cards

Already in the page:
- `<title>`, meta description and keywords, canonical URL (https://campaign.thedailystar.net/syndicates-web/) and robots directives. Large image previews are allowed.
- Open Graph and X (Twitter) cards with a 1200 × 630 crop of the share image (the Korail night aerial).
- `NewsArticle` structured data: headline, description, keywords, author, publish date, and the share image in 16:9, 4:3 and 1:1 crops.
- `sitemap.xml` (the page, its photos and Google News details) and `robots.txt`.

**One step on the live server:** crawlers read `robots.txt` only at the **root of a domain**. `https://campaign.thedailystar.net/robots.txt` doesn't exist today (it returns 404), so choose one:
- create it with the contents of this folder's `robots.txt`; or
- if a root `robots.txt` is added later for other campaigns, add this line to it (don't replace it):

  ```
  Sitemap: https://campaign.thedailystar.net/syndicates-web/sitemap.xml
  ```

**After it's live:**
1. Google Search Console: add `campaign.thedailystar.net` if it's not there, submit `https://campaign.thedailystar.net/syndicates-web/sitemap.xml`, and request indexing of https://campaign.thedailystar.net/syndicates-web/.
2. Check the article data at https://search.google.com/test/rich-results.
3. Refresh the share card at https://developers.facebook.com/tools/debug/ ("Scrape Again").

**Favicon:** `campaign.thedailystar.net` has no favicon today, so the page ships a blank one to avoid a 404. To show The Daily Star's icon, drop `favicon.ico`, `favicon.png` or `favicon.svg` into the project's `assets/brand/` folder and run `npm run release` again.

---

## Folder contents

| Path | What |
|---|---|
| `index.html` | The whole article, prerendered: complete even with JavaScript off |
| `assets/` | Script, styles and fonts (content-hashed names) |
| `media/` | Photos (AVIF, WebP and JPG at several sizes), the hero video (`media/photos/hero-*.mp4`) and the share-card crops in `media/share/` |
| `brand/` | The Daily Star logo, as supplied |
| `robots.txt`, `sitemap.xml` | For search engines (see section 3) |
| `vercel.json` | Vercel preview settings only |
