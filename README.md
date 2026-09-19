# Gadya for Claude Code

Gadya Media's Claude Code plugin: build and look after client websites on [Gadya CMS](https://github.com/gadyamedia/gadya-cms), with the Gadya portal (app.gadya.media) wired in.

| Skill | Use it to |
| --- | --- |
| `/gadya:new-site` | Start a new client site from [gadya-starter](https://github.com/gadyamedia/gadya-starter) and connect it to the portal |
| `/gadya:convert-site` | Move an existing (WordPress or other) site onto Gadya CMS, with redirects |
| `/gadya:migrate-filament-site` | Bring one of our older hand-built Filament sites onto Gadya CMS in place, keeping its own business data |
| `/gadya:upgrade-all-sites` | Bring every site up to the latest gadya/cms, one pull request each |
| `/gadya:morning-check` | See what is down, failing or waiting across all sites and tickets |

## Install

```
/plugin marketplace add gadyamedia/gadya-claude
/plugin install gadya@gadya
```

The portal tools need your personal token. Make one in the portal under **Your account → Security → AI assistants**, then set it before starting Claude Code:

```bash
export GADYA_MCP_TOKEN=gdy_...
# optional, for a local portal:
export GADYA_PORTAL_URL=http://gadya-media.test
```
