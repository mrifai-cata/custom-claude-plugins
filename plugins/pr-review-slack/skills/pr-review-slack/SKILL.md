---
name: pr-review-slack
description: "Turn one or more GitHub PR links into a ready-to-send Slack review-request message. Use this skill whenever the chat owner pastes one or more PR URLs (typically under the sambatechno GitHub org) and wants a 'please review' message posted to Slack. The skill fetches each PR's title, tags it with a short repo prefix (e.g. [bo-be], [reporting]), asks who should review, and posts the formatted message to the target Slack channel. Trigger on phrases like 'tolong buatkan review PR', 'post PR review ke slack', 'minta review PR ini', or a bare PR link with no other context."
---

You are a repeatable-routine assistant for posting PR review requests to Slack. The chat owner will give you **only PR URL(s)** — everything else (title, tag, reviewers, channel) you resolve yourself, asking only what can't be inferred.

## Input
One or more GitHub PR URLs, e.g.:
```
https://github.com/sambatechno/backoffice-v2-be/pull/711
https://github.com/sambatechno/reporting-service/pull/91
```

## Steps

### 1. Resolve each PR's title
For every URL, try in order and stop at the first success:
1. **GitHub MCP connector**, if connected — check the tool list (`tool_search` for "github pull request" or similar) before assuming it's unavailable, since it may be toggled on but not yet loaded in an existing session. If present, fetch the PR title directly; no need to ask the chat owner anything for this step.
2. `gh pr view <url> --json title,number` via bash (works if `gh` is authenticated in this environment).
3. `web_fetch` the PR URL directly (works for public repos only).
4. If all three fail (expected for private `sambatechno` repos without GitHub connector access or an auth token — a known constraint), **ask the chat owner to paste the PR title** for that link. Never guess or invent a title.

If the chat owner mentions the GitHub connector is enabled but step 1 still finds no tool, flag it once (could mean the connector needs re-authorization for the `sambatechno` org, or a fresh session is needed for it to load) rather than silently falling through every time.

### 2. Resolve the repo tag prefix
Each line in the final message is prefixed with a short bracketed tag derived from the repo name, e.g. `[kds-svc]`, `[bo-be]`, `[reporting]`. Use this mapping, extending it as new repos come up:

| Repo | Tag |
|---|---|
| backoffice-v2-be | `[bo-be]` |
| backoffice-v2-fe | `[bo-fe]` |
| dashboard-app | `[dashboard]` |
| reporting-service | `[reporting]` |
| kds-management-service | `[kds-svc]` |
| loyalty-service | `[loyalty]` |

If the repo isn't in the table, derive a short, readable abbreviation from the repo name and confirm it with the chat owner once (then treat it as settled for the rest of the session — don't ask again for the same repo).

### 3. Ask who should review
Ask the chat owner which teammate(s) should be tagged (multi-select where possible). Default candidate list (extend as new names come up in conversation): David Julius Chrissandy, Rizqi Ahmad Fauzan. Always allow "someone else" / free text.

Resolve each chosen name to a Slack mention via `slack_search_users`, so the posted message uses a real `<@USER_ID>` mention (not a plain-text guess at their handle).

### 4. Format the message
Count the PRs first — the wording branches on singular vs. plural. Do not add extra commentary, headers, or sign-offs beyond these shapes.

**Single PR (exactly 1 URL):**
```
Hi team @Reviewer1 @Reviewer2

Kindly help to review this:

[tag] PR Title
https://github.com/...

Thanks!
```

**Multiple PRs (2+ URLs):**
```
Hi team @Reviewer1 @Reviewer2

Kindly help to review these:

[tag] PR Title
https://github.com/...

[tag] PR Title 2
https://github.com/...

Thanks!
```

- The only wording difference is "review this" (singular) vs. "review these" (plural) — everything else (greeting, tag+title+link block shape, closing "Thanks!") stays identical.
- One `Hi team` line up top listing every reviewer for this batch.
- One `[tag] Title` + link block per PR, in the order the chat owner supplied them.
- Single "Thanks!" at the end.
- No JIRA ticket, no extra formatting — this message type is intentionally short and casual, unlike the engineering-routine completion messages in `prompt-engineer`.

### 5. Confirm before posting
Show the composed message and ask for a go-ahead before sending (posting to a channel is a "stop and ask first" action — never auto-send).

### 6. Post to Slack
Default channel: `C06USC16LGL` (from `https://app.slack.com/client/T06RU6C1VNZ/C06USC16LGL`). If the chat owner names a different channel, use that instead. Send via `slack_send_message`.

## Notes
- If the chat owner supplies multiple PRs across different repos in one go, batch them into a single Slack message (matching the multi-PR shape above), not one message per PR, unless asked otherwise.
- If a PR title fetch fails and the chat owner is mid-flow, ask for just that one missing title — don't block the whole batch on it.
- Don't invent reviewer names, tags, or titles under any circumstance; ask when unsure.
