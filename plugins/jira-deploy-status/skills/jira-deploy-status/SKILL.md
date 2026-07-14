---
name: jira-deploy-status
description: >-
  Post a proactive deploy-status update on a JIRA ticket. Given only a JIRA
  issue key (e.g. GRO-133), it looks up the ticket, finds the related merged/open
  pull requests across the sambatechno repos, works out how far the change has
  travelled (merged to develop → staging → production), drafts an English status
  comment, and — after you approve it — posts the comment to the ticket. Use when
  the user says things like "be proactive on <TICKET>", "update <TICKET> with
  deploy status", "comment on <TICKET> that it's merged", or wants a recurring
  deploy-status update on a ticket.
---

# JIRA deploy-status update

Purpose: turn a bare JIRA ticket key into an accurate, English deploy-status
comment on that ticket. This automates the "let stakeholders know the work is
merged and where it is in the release pipeline" step.

## Input

A single JIRA issue key, e.g. `GRO-133`. Nothing else is required. If the user
gives extra hints (target env, "already on staging"), fold them in.

## Prerequisites (tools)

- **Atlassian MCP** — `getJiraIssue`, `searchJiraIssuesUsingJql`,
  `addCommentToJiraIssue`, `getTransitionsForJiraIssue`, `transitionJiraIssue`.
  Get the `cloudId` from `getAccessibleAtlassianResources` (the CATA site is
  `catasg.atlassian.net`).
- **GitHub MCP** — `search_pull_requests`, `pull_request_read`, `get_commit`.

Load deferred MCP tool schemas with `ToolSearch` (`select:<tool>`) before calling.

## Procedure

1. **Read the ticket.** `getJiraIssue` for the key with fields
   `summary, status, assignee, description, comment`. Note the current status
   (e.g. `QA Passed`, `In Progress`) — it drives the wording.

2. **Find the PRs.** For each candidate repo, run
   `search_pull_requests` with `repo:sambatechno/<repo> <KEY>`. Candidate repos:
   `backoffice-v2-fe`, `backoffice-v2-be`, `dashboard-app`, `reporting-service`,
   `loyalty-service`, `promo-service`, `samba-mobileapp`, `cata-airflow`.
   - Match the ticket key in the PR title/body; ignore old PRs that merely list
     the key among many others unless clearly relevant.
   - For each relevant PR capture: number, title, `state`, `merged_at`, and base
     branch. A PR with `merged_at` set and base `develop` = merged to develop.

3. **Determine the pipeline stage.** Map what you found to one of:
   - *Open / in review* — PR(s) not yet merged.
   - *Merged to `develop`* — merged, base develop, not yet on staging.
   - *On staging* — the user said so, or a release/tag/deploy confirms it.
   - *On production* — likewise.
   Only claim staging/production if there is evidence (user statement, release
   tag, deploy workflow run). Do **not** assume a merge to develop means it is
   live. When unsure, say "pending release to staging".

4. **Draft the comment (English).** Keep it factual and short. Template:

   ```
   Deploy status — <stage summary>

   Implementation for <KEY> is <complete/merged>. Status: <JIRA status>.

   Merged PRs:
   - FE — sambatechno/backoffice-v2-fe#<n> · <title>
   - BE — sambatechno/backoffice-v2-be#<n> · <title>

   What changed: <one/two sentence functional summary from PR bodies>.

   Next: <what's pending — e.g. awaiting release train to staging, then production>.
   I'll update this ticket once it's live on each environment.
   ```

   - Always English (matches the CATA ticket-thread convention), regardless of
     the language the user prompts in.
   - Reference PRs as `owner/repo#number` so JIRA renders them as links.
   - Do not paste internal hostnames, credentials, or env-var values.

5. **Show, then post.** Present the draft to the user and wait for approval
   ("ACC"/"ok") **before** calling `addCommentToJiraIssue`. If they ask for
   edits, revise and re-show. Post with `contentFormat: "markdown"`.

6. **(Optional) Transition status.** Only if the user asks. Call
   `getTransitionsForJiraIssue` first to get valid transition IDs, then
   `transitionJiraIssue`. Never guess a transition that isn't in the list.

## Follow-up mode (recurring)

If the user wants ongoing updates ("keep this ticket posted as it moves to
staging/prod"), use the `/loop` skill or `send_later` to re-run this procedure
on an interval, and post a new comment only when the pipeline stage actually
changes — never spam duplicate comments.

## Guardrails

- **Approval before posting.** Comments and transitions are outward-facing; get
  an explicit go-ahead first.
- **Accuracy over optimism.** If PRs aren't merged, say so. If you can't confirm
  staging/prod, say "pending" rather than claiming it's live.
- **No duplicate comments.** Check existing ticket comments; if the same status
  was already posted, skip or update instead of re-posting.
- **Scope.** Only search the sambatechno repos listed above.
