---
name: monday-ads-report
description: Use when someone asks for the weekly ads report, the Monday report, "how did my Google and Meta ads do last week", or a week-over-week and month-to-date review of their ad accounts. Pulls Google Ads and Meta Ads through their MCP servers, compares last week with the week before and this month with last month, ranks campaigns and creatives, and ends with specific actions tied to numbers. Read-only, changes nothing in an ad account.
---

# Monday ads report

The weekly report I send every Monday for the ad accounts I run. It has gone out on 14
Mondays since August 2026, and every rule below is here because a report once got it wrong.

The report answers four questions, in this order:
1. How did last week do against the week before?
2. Where is the month heading against last month?
3. What worked (campaigns, creatives) and what is fatiguing?
4. What should I do this week, and roughly what is it worth?

## Needs

- **Meta Ads MCP** (Meta's official server, tool `ads_get_ad_entities`), and/or
- **Google Ads MCP** (Google's official server, tool `search` / `search_search`).
- The account ids for each report. Ask once and keep them in a `reports.json` beside you:
  `[{"name": "Shop A", "meta": "act_123", "google": "1234567890", "campaign_filter": null}]`.

One channel missing is fine. Say so in the report and do the other.

## Steps

1. **Work out the windows.** All windows are inclusive and end yesterday at the latest.
   Never include today: a half day looks like a crash.
   | Window | Definition |
   |---|---|
   | `LAST_WEEK` | the most recent complete Monday to Sunday |
   | `PREV_WEEK` | the Monday to Sunday before it |
   | `MTD` | 1st of this month to yesterday |
   | `PREV_MTD` | 1st of last month to the same day of month (like for like) |
   If yesterday was the last day of a month, MTD is that whole month. Say so.

2. **Pull Meta, campaign level, for all four windows.**
   `ads_get_ad_entities(level: "campaign", fields: ["id","name","effective_status","objective","amount_spent","impressions","clicks","reach","frequency","cpm","ctr","purchase_roas","omni_purchase_values","actions:omni_purchase","cost_per_omni_purchase"], time_range: {...}, sort: "amount_spent_descending", limit: 50)`
   - Field names are bare (`amount_spent`), never prefixed (`campaign.amount_spent`).
   - A campaign with no delivery comes back with its metric keys **missing**, not zero.
     Treat missing as zero delivery and leave it out of the tables.

3. **Pull Meta, ad level, for `LAST_WEEK` and `MTD`.** Same call with `level: "ad"` and add
   `video_p75_watched_actions`, `outbound_clicks_ctr`. Pull it twice: sorted by spend and sorted
   by `purchase_roas_descending`. Spend order alone hides the small winners.
   - Do **not** ask for `post_shares` at ad level. It throws a validation error and takes the
     whole pull down with it.

4. **Pull Google, campaign level, for all four windows.**
   ```
   SELECT campaign.id, campaign.name, campaign.status, campaign.advertising_channel_type,
          metrics.cost_micros, metrics.impressions, metrics.clicks, metrics.conversions,
          metrics.conversions_value, metrics.ctr, metrics.average_cpc
   FROM campaign
   WHERE segments.date BETWEEN 'START' AND 'END' AND metrics.impressions > 0
   ORDER BY metrics.cost_micros DESC
   ```
   - Cost is in micros. Divide by 1,000,000.
   - GAQL has no `OR`. Run separate queries.
   - **Never put a low LIMIT on an "all campaigns" query.** One account had 684 campaigns, a
     LIMIT 300 cut off a whole market, and the report said that market had zero campaigns.
     If a sweep returns a surprising zero, check the limit before you write a conclusion.
   - `metrics.search_impression_share` only works for Search campaigns. Query it separately so
     one bad field cannot lose the whole pull.

5. **Apply any `campaign_filter` after fetching**, in your own analysis, never in the API call,
   so nothing disappears without a trace.

6. **Check the whole chain before calling anything "live".** An ad group, asset group or ad can
   be ENABLED inside a PAUSED campaign. It serves nothing. A report once said two campaigns
   were eating budget. Both were paused and had zero delivery. Before any claim that something
   is live, wasting spend or should be paused, confirm: the campaign is enabled, the object's
   `primary_status` is eligible, and it had impressions in the window.

7. **Analyse.**
   - **Last week vs the week before:** spend, impressions, clicks, CTR, purchases or
     conversions, revenue, ROAS, CPA. Meta and Google separately, then combined.
   - **Month to date vs last month to date:** the same set.
   - **Best campaigns:** top 5 by spend and top 5 by ROAS, with at least 3 conversions so noise
     can't win. Name the channel on every row.
   - **Creatives (Meta):** top 5 by ROAS and by spend for last week. Flag frequency above 3.0
     as fatiguing, and CTR below half the account average as a cut candidate.

8. **Write the actions.** This is the part with the value. Look for: budget on campaigns below
   1.0 ROAS (or below the breakeven ROAS if the user gives you margin), creative fatigue,
   Search losing impression share to budget vs rank, Performance Max with stale assets, and
   broken tracking (spend with zero conversions where there used to be some). Every action
   gets **the trigger, the action and the money it is worth**. If you can't tie it to a number
   in the data, cut it.

## Rules

- Never invent a number. Every figure comes from a pull you made in this run.
- If a pull fails, put a visible **Data gap** box in the report: the account, what you tried,
  the error as returned. Never quietly skip a channel.
- Platform revenue is what the platform claims. Meta and Google each count every sale they
  touched, so their totals add up to more than the shop sold. Say "platform-reported".
- Nothing changes in an ad account. This skill only reads.
- A short, honest report beats a padded one.

## Output format

1. **Headline** (3 lines, readable on a phone): the result, the number that moved most, the one
   action for this week.
2. **Last week vs previous week:** KPI table with % change.
3. **Month to date vs last month:** KPI table with % change.
4. **Best campaigns** and **best creatives:** tables.
5. **Actions:** numbered, each with trigger, action, estimated money.
6. **Data gaps** (if any).

## Verification checklist

- [ ] No window includes today.
- [ ] Meta campaigns with missing metrics were treated as zero delivery, not errors.
- [ ] No Google query had a LIMIT that could cut off campaigns.
- [ ] Every "live" or "wasting spend" claim passed the chain check in step 6.
- [ ] Every action has a trigger, an action and a money figure.
- [ ] Every failed pull shows as a Data gap.
