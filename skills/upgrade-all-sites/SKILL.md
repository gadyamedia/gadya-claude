---
name: upgrade-all-sites
description: Bring every Gadya CMS client site up to the latest gadya/cms - find which are behind through the Gadya portal, run the gadya-cms-upgrade skill in each project on a branch, and open a pull request per site. Use when asked to upgrade, update or patch all sites or the fleet.
---

# Upgrade every site

Uses the Gadya portal (MCP tools `list-sites-tool`, `get-site-tool`) to see what each site runs, and each project's own `gadya-cms-upgrade` skill to do the work. It opens one pull request per site and never merges: merging deploys.

## 1. Which sites are behind

1. The latest release: `curl -s https://repo.packagist.org/p2/gadya/cms.json | php -r '$d=json_decode(stream_get_contents(STDIN),true); echo $d["packages"]["gadya/cms"][0]["version"], PHP_EOL;'`
2. `list-sites-tool`: every site with its `gadya_cms` version. Behind = older than the latest. Also note sites with `cms_todo` > 0: they are on the latest but have not taken up everything.
3. Show the person the list and confirm which to do.

## 2. Find each project

A site's code lives in a local folder whose `composer.json` requires `gadya/cms` and whose `.env` `APP_URL` host (or `config/gadya-cms.php` `seo.domains`) matches the site. To find them:
- search the usual places (`~/Sites`, `~/Sites/client-work`) to depth 3;
- remember what you find in `~/.config/gadya/sites.json` (`{"<site host>": "<absolute path>"}`) so the next run is instant.

A site with no local folder: clone it with `gh repo clone gadyamedia/<repo>` once the person names the repository.

## 3. Upgrade each one

For each site, one at a time, in its folder:

```bash
git fetch && git switch main && git pull
git switch -c chore/gadya-cms-<version>
```

Follow that project's `gadya-cms-upgrade` skill end to end: update, migrate, apply the upgrade notes, work down `gadya-cms:audit`, and make the tests pass. Then:

```bash
git push -u origin HEAD
gh pr create --title "chore: gadya/cms <old> → <new>" --body "<what changed, audit before/after, anything for the person to do>"
```

When a site's tests fail for a reason the upgrade did not cause, stop on that site, note it, and move on.

## 4. Report

End with a table: site, old → new version, audit to-dos before → after, PR link, and anything that needs a person, such as a failing test or a config decision. Merging each PR deploys that site.
