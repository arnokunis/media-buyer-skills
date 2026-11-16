# Media buyer skills for Claude

Claude skills I use for the ad accounts I run every week. Each one is a folder with a
`SKILL.md`. Claude opens it on its own when the task matches.

| Skill | What it does |
|---|---|
| [monday-ads-report](skills/monday-ads-report/SKILL.md) | Last week vs the week before, month to date vs last month, Google + Meta, best campaigns and creatives, and actions tied to numbers. Read-only. |

More coming: budget pacing, search-terms cleaner, weekly DSA builder.

## Install

**Claude Code:** copy a skill folder into `~/.claude/skills/`.

**Claude desktop / claude.ai:** zip the skill folder, then Settings → Capabilities → Skills → Upload.

You also need the ad platforms connected: Meta's official Ads MCP and/or Google's
official Google Ads MCP.

## Ground rules in every skill

- Never invents a number. Every figure comes from a real pull.
- A failed pull shows as a data gap. It never gets skipped.
- Reads only. Nothing changes in your ad account.

MIT licensed. Use them, change them, share them.
