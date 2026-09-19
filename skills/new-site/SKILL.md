---
name: new-site
description: Start a new Gadya Media client website from the gadya-starter template - interview, create the repo, install Laravel + Filament + Gadya CMS, apply the brand and pages, connect it to the Gadya portal, and hand over hosting steps. Use when asked to build, start, spin up or set up a new client site.
---

# New client site

Builds a working, tested site on [gadya-starter](https://github.com/gadyamedia/gadya-starter), then connects it to the Gadya portal so it is monitored and supported from day one.

## 1. Interview

Ask only for what you cannot find. Take the rest from the brief, an existing site, or sensible defaults. Use AskUserQuestion for choices.

- **Business:** name, what it does, where, phone, email, address, social links.
- **Client in the portal:** its exact name. `connect-site-tool` needs it to exist; if it does not, ask the person to create it in the portal first.
- **Domains:** the live domain first, then any others that should redirect.
- **Starting point:**
  - nothing yet;
  - an existing website (then use the `convert-site` skill for the content, after step 3);
  - a design (a Figma link or reference sites).
- **Brand:** primary, secondary, background, ink and accent colours, and a display font and a body font. With a logo or a site to go on, derive them and confirm.
- **Pages:** home plus the rest (services, pricing, about, contact, locations...). Also whether to include the blog, events (What's on), comments and the newsletter.
- **Hosting:** Laravel Forge or Laravel Cloud, and the GitHub organisation (default `gadyamedia`).

## 2. Create the repository

```bash
gh repo create gadyamedia/<slug> --private --template gadyamedia/gadya-starter --clone
cd <slug>
composer install && npm install
cp .env.example .env && php artisan key:generate
```

Set in `.env`:
- `APP_NAME`, `APP_URL` (the local Herd address, `http://<slug>.test`);
- the `BRAND_*` colours and fonts (`BRAND_FONT_STYLESHEET` for Bunny or Google Fonts);
- `SITE_ENQUIRIES_EMAIL`.

## 3. The content

Rewrite `config/site.php` for this business; it is the site document the CMS seeds from:
- every page from the interview, with a real heading, description and SEO snippet (`seo.meta_title`, `seo.meta_description`);
- the menu (`nav`) and the footer menu (`menus.footer`);
- `phone`, `address`, the footer tagline and the announcement.

No placeholder copy may survive: search for "Springfield" and "555", and there must be none left.

Then:

```bash
php artisan migrate
php artisan gadya-cms:install --admin-name="<client contact>" --admin-email="<email>" --admin-password="<generated>"
php artisan boost:install
```

Generate the admin password with `php -r 'echo bin2hex(random_bytes(9));'`, and give it to the person at the end, never in a commit.

## 4. The design

Restyle `resources/views/layouts/site.blade.php`, `resources/views/pages/show.blade.php` and `resources/css/app.css` to the brand and any design reference. Keep every directive:
- `@cmsSeo`, `@editable…`, `@siteImage`;
- the search and newsletter forms;
- `@gadyaBuiltBy` last in the footer;
- `@cmsToolbar`.

Add page types with `php artisan gadya-cms:make:page-template <type>`, and list every new editable path in `gadya-cms.editable_fields`. Follow the `gadya-cms-development` skill that `boost:install` put in the project.

Turn off what the interview left out (`->events(false)`, `blog.comments.enabled`...). Then run:

```bash
npm run build
php artisan gadya-cms:audit     # nothing left to do, or a reason for each optional item
php artisan test --compact
```

Look at the site at its `.test` address and in the admin before going on.

## 5. Connect it to the Gadya portal

1. Call the MCP tool `connect-site-tool` with the client name and the **live** URL. It returns a pairing code.
2. Install the connection and store the code for launch:

```bash
composer require gadya/connect
php artisan migrate
```

The code is for the live server: run `php artisan gadya:connect <code>` there after the first deploy (it lasts 48 hours), or get a new one from the site's page in the portal.

## 6. Commit and hand over

Commit (`feat: <client> website on gadya-starter`) and push. Tell the person:

1. The repo, the local address, and the admin sign-in (email and the generated password).
2. **Forge:**
   - create the site on the server, the domain first, then aliases for the other domains with a redirect to it;
   - connect the repository;
   - paste the deploy script from the README;
   - enable Quick Deploy;
   - add the scheduler and a queue worker;
   - issue the Let's Encrypt certificate;
   - delete the `location = /robots.txt` line in the nginx config.

   **Laravel Cloud:** create the application from the repository, add the domain, a database and a queue worker; the scheduler is built in.
3. After the first deploy: `php artisan gadya:connect <code>` on the server.
4. **DNS and Search Console:** follow **Settings → Get found** in the site's admin for the exact records and the sitemap.
5. Keys the client enters in the admin: AI under Settings → AI, Google under Settings → Search & speed.
