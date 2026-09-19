---
name: convert-site
description: Move an existing website (WordPress or any other) onto Gadya CMS - pages into the site document, posts into articles, images into the photo library, and a redirect for every old address. Use when a client already has a site and it is being rebuilt on Gadya CMS.
---

# Convert an existing site

Use it inside a project already made with the `new-site` skill (or any Gadya CMS site). The client's words, pictures and search rankings must survive the move.

## Rules

- **Read-only on the old site.** Never log in to it or change it.
- Copy the content faithfully. Tidy obvious mistakes, but do not rewrite the client's copy unless asked.
- Every old address that had traffic or links gets a redirect. A rebuilt site that 404s its old pages loses its Google ranking.
- Images go in through the library, never hot-linked.

## 1. Inventory

- `curl -s <old>/sitemap.xml` (and `/sitemap_index.xml`, `/wp-sitemap.xml`): every page and post address.
- **WordPress:** the REST API gives clean content:
  - `/wp-json/wp/v2/pages?per_page=100`
  - `/wp-json/wp/v2/posts?per_page=100&_embed`
  - `/wp-json/wp/v2/categories`
  - `/wp-json/wp/v2/tags`
  - `/wp-json/wp/v2/media?per_page=100`
- **Anything else:** fetch each page and take the main content (headings, paragraphs, lists, images), leaving out the header, footer and navigation.

Show the person the inventory (pages, posts, images, forms) and agree what to keep before importing.

## 2. Pages → the site document

For each page, add an entry to `config/site.php` (before the first `gadya-cms:install`), or to the draft through `SiteContentRepository::saveDraft()` once installed:
- slug from the old path;
- title, heading, description, sections in the page type's shape;
- `seo.meta_title` and `seo.meta_description` from the old page's title tag and meta description.

Rebuild the menu (`nav`) from the old one.

## 3. Posts → articles

Create `Gadya\Cms\Models\Post` rows through a one-off artisan command, which you delete afterwards:
- title, slug (keep the old one), HTML body, excerpt, published date;
- author name, the categories and tags as terms, the featured image.

Keep `blog.prefix` matching the old URLs where you can (e.g. `blog`). Otherwise add redirects.

## 4. Images

Download every image the kept content uses into `public/images/site/` (keep the file names), then run:

```bash
php artisan gadya-cms:import-legacy-media
php artisan gadya-cms:media-variants
```

Point the content at the library file names (`@siteImage('name.jpg')`).

## 5. Redirects

For every old address whose path changes, create a redirect: **Settings → Redirects** in the admin, or `Gadya\Cms\Models\Redirect` rows. Check each old URL against the new site: every one must answer 200 or redirect once to a 200.

## 6. Check and hand over

```bash
php artisan gadya-cms:audit
php artisan gadya-cms:agent-ready
php artisan test --compact
```

Report to the person:
- how many pages, posts and images were imported, and how many redirects were made;
- anything that did not fit, such as contact-form plugins, shop pages or members' areas.

The client then reviews the draft and presses Publish.
