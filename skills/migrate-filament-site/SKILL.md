---
name: migrate-filament-site
description: Bring an existing Laravel + Filament site that we built without Gadya CMS onto Gadya CMS, in place - the page words, settings, menu, photos, SEO and redirects move into the site document and the client edits them on the page, while the site's own business data (products, rentals, services, jobs) stays in its own Filament resources. Use for our older hand-built Filament sites (gadya, citybounce, pml, ghweb and the like), not for WordPress or other outside sites (that is convert-site).
---

# Migrate a Filament site onto Gadya CMS

The site keeps its code, its database and its URLs. Only the parts Gadya CMS does better move into it: words on pages, settings such as phone and address, the menu, photos, SEO tags, redirects, enquiries and analytics. The client then edits on the real page and presses **Publish changes**.

Documentation for the installed version is in `vendor/gadya/cms/docs/` once it is required; read the file named in each step before changing what it covers.

## Rules

- **Branch first, never push.** Start from a clean tree: `git switch -c feat/gadya-cms`. Pushing may deploy; leave that to the person.
- **Never touch production data from here.** Existing tables stay as they are. Content moves through a one-off command that the person runs after deploying (step 6). It only writes the draft, and can be run again.
- **Never run** `migrate:fresh`, `db:seed` or `gadya-cms:install --force` on a real database.
- **Every public URL keeps answering.** Record each address before you start and check each one again at the end. A URL that must change gets a redirect.
- **Keep the business data where it is.** Rentals, packages, lab tests, job postings and orders stay in their own models and Filament resources. Templates read them as before. Only the words around them become editable.
- **Business facts come from the site.** Names, phones and addresses come from its settings tables, `.env` and templates, never from the package's examples.
- Follow the project's own `.ai/rules`, CLAUDE.md and AGENTS.md. Run Pint on PHP you touch, and run the tests after every step.

## 1. Survey

```bash
git status --short                       # must be empty
composer show --direct | grep -E 'laravel/framework|filament/filament|livewire/livewire|intervention/image'
php -v | head -1
ls app/Models app/Filament/Resources app/Filament/Pages
php artisan route:list --except-vendor
php artisan test --compact               # the baseline; note what already fails
```

Save every public GET address, including the dynamic ones filled from the database, to a scratch file. You'll check them again in step 8. Where the site has a sitemap, `curl -s <live>/sitemap.xml` is the quickest list.

Sort every model into one of three kinds:

| Kind | Examples we have | Where it goes |
| --- | --- | --- |
| **Page words and settings** | `SiteSetting`, `SeoSetting`, `SeoMeta`, `HeroSlide`, `Faq`, `Testimonial`, `Promo`, a `Page` whose rows are marketing pages | the site document (`pages.*`, `site.*`) |
| **Things Gadya CMS already has** | `MenuItem` / `NavigationMenu` / `NavigationItem`, `Media`, `Redirect`, `ContactSubmission` / `ContactForm`, `PageView` / `AnalyticsEvent`, `Post` / `ArticleGeneration`, `UserInvitation`, `WaitlistSignup` | the package's own feature (menu, photo library, redirects, forms inbox, analytics, articles, team, newsletter) |
| **Business data** | `RentalItem`, `Package`, `Category`, `ServiceArea`, `LabTest`, `Location`, `JobPosting`, `Certificate`, `PortfolioItem`, `PaymentRate` | stays: its model, resource and templates are unchanged |

Show the person this table for their site and agree it before building. Two kinds of thing are theirs to decide:
- a model that could go either way, such as FAQs a client edits once a year versus FAQs attached to products;
- a custom feature that Gadya CMS has in a different shape, such as a chat inbox, rank tracking or audit logs. Those stay as they are.

## 2. Bring the stack up to the package's requirements

gadya/cms needs PHP 8.3+, Laravel 13, Filament 5, Livewire 4 and intervention/image 3. Upgrade anything below that first, as its own commit, with the tests passing:
- Laravel 12 → 13: follow the official upgrade guide.
- Filament 4 → 5 (and Livewire 3 → 4): `composer require filament/upgrade:"^5.0" -W --dev`, then `vendor/bin/filament-v5`, then follow its output.

Use `search-docs` (Laravel Boost) for each upgrade guide rather than memory.

## 3. Require and install

```bash
composer require gadya/cms -W
```

Before `gadya-cms:install`, write `config/site.php`, the first site document. Read `docs/site-document.md` first.
- `site`: name, phone, email, address, hours and socials, read from the settings tables or the current templates.
- `pages`: one entry per marketing page, with the words the templates show today, copied faithfully. Each entry also gets its `seo.meta_title` and `seo.meta_description` from the current tags.
- `nav`: the current menu.

Build this array locally from the project's own seed data or templates. Real client content arrives in step 6.

Then:

```bash
php artisan gadya-cms:install --no-admin
```

It publishes the config, runs the package's migrations (every table is `gadyacms_*`, so nothing clashes with the site's own), seeds the document and indexes `public/images/site`.

## 4. The panel

Read `docs/installation.md` and `docs/roles-and-globals.md`.

- Add `GadyaCmsPlugin::make()` to the existing panel, next to the site's own resources, and `->passwordReset()`.
- **Dashboard:** the package's dashboard sits at the panel root. Remove the site's own `Dashboard` page and keep its widgets by registering them on the panel.
- **Users:**
  - Give the user model a `role` column: add a migration, and default existing users to the admin role, because they are the people who manage the site today.
  - Implement `FilamentUser`, and define the `manage-content` and `manage-users` gates.
  - Set `users.roles`, `default_role` and `admin_role` in `config/gadya-cms.php`. Keep `role` out of `$fillable`.
- **Duplicates:** remove the site's own resources for anything that moved into the package: menu, media, redirects, SEO settings, contact submissions, invitations, analytics pages. Keep their models and tables until the client has signed off (see step 9).
- **Switches:** for features the site doesn't want, set `->blog(false)`, `->events(false)` and so on, rather than leaving them empty.

## 5. Templates

Read `docs/live-editor.md` and `docs/site-document.md`. Then go page by page:

- **Controller:**
  - Resolve the document with `SiteContentRepository::forRequest()` and `PublicDocument::from()`, calling `EditContext::boot()` first.
  - Pass the document's `$page` and `$site` alongside the business data the controller already loads.
  - 404 when `PageRegistry::isHidden($page)`, unless `EditContext::showsDraft()`.
- **Template:**
  - Replace hard-coded words and settings reads with the document: `$page['heading']`, `$site['phone']`.
  - Mark each one `@editable(...)` inside `@editableFor("pages.{$slug}")`.
  - Add every path to `gadya-cms.editable_fields`.
- **Photos:**
  - `@siteImage($page['hero_image'], 960)` with `@siteSrcset(...)`.
  - Copy every photo the site serves into `public/images/site/` (from `storage/app/public` or wherever the old `Media` model stored them), keeping the file names.
- **Layout:**
  - Replace hand-written title, description, canonical and Open Graph tags with `@cmsSeo($page)`.
  - Add `@cmsToolbar` before `</body>` and `@gadyaBuiltBy` at the end of the footer.
- **Forms:** send enquiry forms through `@cmsForm('key')` and `@cmsFormStatus('key')` (`docs/forms.md`).
- **Old sitemap:** delete the hand-made sitemap route and view (`sitemap.blade.php`), a static `public/robots.txt`, and any spatie/laravel-sitemap wiring. Then list the business data's addresses in the package's sitemap with `SitemapEntries::add(fn () => …)` in a service provider (gadya/cms 0.5.4+, *Sitemap and robots* in `docs/seo.md`).
- **Old dynamic pages** (such as a `Page` model served at `/{slug}`): move them into `pages.*`. Then either let the package's page route serve them, or keep the controller and read the document; `pages.route_excluded_slugs` settles the conflict.

Write a feature test per template, in the project's style, that publishes a document and asserts the page shows its words. Use the `publishDocument([...])`-style helpers from `docs/site-document.md`, and keep the tests that already cover the business data passing.

## 6. The one-off content transfer

The client's live words are in the production database, not in the repo. Write `app/Console/Commands/MoveContentToGadyaCms.php` (`gadya:move-content`). It:

- reads the old tables (`site_settings`, `seo_settings`, `pages`, `hero_slides`, `faqs`, `menu_items`, `redirects`, `contact_submissions`, …) and maps each value to its document path;
- writes it into the draft through `SiteContentRepository::saveDraft()`. It never publishes, and it never writes to the old tables;
- creates `Gadya\Cms\Models\Redirect` rows from the old redirects table;
- copies old uploads into `public/images/site/`, then calls `gadya-cms:import-legacy-media` and `gadya-cms:media-variants`;
- prints what it moved and what it skipped. Running it again gives the same result.

Test it: seed the old tables with factories, run the command, and assert the draft holds the values and the old rows are untouched.

## 7. Audit

```bash
php artisan gadya-cms:audit              # clear every "!", decide every "○" (the upgrade skill's step 5)
vendor/bin/pint --dirty --format agent
php artisan test --compact
php artisan gadya-cms:agent-ready
```

## 8. Check every address

Request every address from step 1, through the app, with `get-absolute-url` and `curl -s -o /dev/null -w '%{http_code}'`. Each one must answer 200, or redirect once to a 200. Also check `/sitemap.xml`, `/robots.txt` and `/llms.txt`.

## 9. Commit and hand over

Commit on the branch, in steps: stack upgrade, package install, panel, templates, transfer command. Don't push.

Tell the person:
1. **The map:** what moved into Gadya CMS, what stayed as the site's own data, and anything left for them to decide.
2. **Deploy order:**
   1. Deploy; the deploy script must run `migrate --force` and `filament:assets`.
   2. Run `php artisan gadya:move-content` on the server.
   3. Open the live editor and check the draft against the old live site.
   4. **Publish changes.**
   5. Make sure the cron runs `schedule:run` every minute.
3. **Pairing:** pair with the portal (`php artisan gadya:connect GDY-…`).
4. **Later, in a separate PR once the client has used it for a few weeks:** drop the old tables and models that moved, and delete the transfer command.
