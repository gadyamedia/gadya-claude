---
name: morning-check
description: A quick look across every Gadya Media client site and the support queue - what is down, failing, silent, expiring or waiting on us - with the next action for each. Use when asked what needs attention, for a daily check, or how the sites are doing.
---

# Morning check

Everything comes from the Gadya portal's MCP tools. Read only: this skill changes nothing.

1. `list-sites-tool` with `needs_attention: true`: the sites that are down, failing their health checks, not checking in, or within 14 days of a certificate expiring.
2. For each, `get-site-tool`; for a down or failing one also `site-logs-tool`, to find the cause (the last errors, a failed deploy, a full disk, a stopped queue).
3. `list-tickets-tool`: open tickets. Pick out the urgent ones, those with no reply for more than a working day, and those unassigned.
4. Report in this order, one line each, with the portal link:
   - **Down or failing now:** the cause as far as the logs show, and the next step.
   - **Tickets waiting on us:** oldest first.
   - **Coming up:** certificates expiring, sites behind on gadya/cms (see `upgrade-all-sites`), silent sites.
   - **All clear:** a count of the healthy sites.

Offer the next actions (reply to a ticket, deploy, look at a log) but take none without a yes.
