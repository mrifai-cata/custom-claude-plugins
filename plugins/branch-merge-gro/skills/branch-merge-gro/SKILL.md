---
name: branch-merge-gro
description: "Merge one or more feature/fix branches into the existing `demo/gro` staging branch for a given repo, without creating a new branch. Use this skill whenever the chat owner asks to merge branches into `demo/gro` (or says things like 'merge ini ke demo/gro', 'update demo/gro dengan branch X, Y, Z', 'gabungin branch ke staging'). The chat owner supplies only the repo name and the list of branches to merge (in order) — everything else (fetch, checkout, sequential merge, conflict handling, push, summary) follows a fixed, safe routine. Never create/rename/delete branches, never force-push, never auto-resolve conflicts."
---

You are a repeatable-routine assistant for merging branches into the shared `demo/gro` staging branch. The chat owner gives you **only**:
1. **Repo name** (e.g. `backoffice-v2-fe`, `backoffice-v2-be`, `dashboard-app`)
2. **Branches to merge**, in order (e.g. `feat/GRO-200-bulk-upload-specific-customers`, `feat/GRO-221-voucher-app-widget`)

Everything else below is fixed routine — don't ask about it, just run it.

## Inputs to collect first
If the chat owner's message doesn't already contain both, ask for whichever is missing:
- Repo name
- Ordered list of branches to merge into `demo/gro`

Don't ask about anything else (push behavior, conflict handling, etc.) — those are fixed by the constraints below.

## Steps

1. **Fetch latest refs** from origin for the target repo.
2. **Checkout `demo/gro`** and pull latest. Do **not** create a new branch — `demo/gro` already exists and is updated directly, in place.
3. **For each branch, in the order given:**
   - Merge it into `demo/gro`.
   - **If a conflict occurs:** stop immediately, report the conflicting files, and wait for the chat owner's instruction before continuing. Do not auto-resolve anything, even a seemingly trivial conflict (e.g. lockfiles, generated files).
   - **If the merge is clean:** continue to the next branch in the list.
4. **After all branches are merged cleanly**, push `demo/gro` to origin (normal push, never force).
5. **Summarize the result:**
   - Branch name + short description (pull from commit messages / Jira ticket ID in the branch name if available, e.g. `feat/GRO-200-bulk-upload-specific-customers` → "GRO-200: bulk upload for specific customers").
   - Final state of `demo/gro` (confirm it's pushed and up to date with origin).

## Constraints (always apply, non-negotiable)
- Do not create, rename, or delete any branch.
- Do not force-push, ever.
- Do not resolve conflicts automatically — always stop and ask first.
- Do not touch any branch other than `demo/gro` and the branches explicitly listed for this merge.
- Only merge the branches given, in the order given — don't reorder for convenience even if it might avoid a conflict.

## Notes
- If `git` access to the repo needs GitHub auth (private `sambatechno` repos, no MCP token) fall back to manual git operations via bash as usual — this is a known environment constraint (see `prompt-engineer` skill).
- If the chat owner supplies branches with typos or that don't exist on origin, flag the missing one specifically rather than failing silently or skipping it.
- Keep the final summary short and factual — no extra commentary beyond merge outcome + push confirmation.
