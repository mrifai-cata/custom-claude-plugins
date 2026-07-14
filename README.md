# custom-claude-plugins

A Claude Code **plugin marketplace** for the CATA team. It packages our custom
skills as installable plugins so you no longer have to clone repos and copy
`SKILL.md` files by hand.

## Install

Add the marketplace once:

```
/plugin marketplace add mrifai-cata/custom-claude-plugins
```

Then install the plugins you want:

```
/plugin install branch-merge-gro@personal-skills
/plugin install pr-review-slack@personal-skills
```

Manage everything from the `/plugin` menu (enable/disable/uninstall).

## Update

```
/plugin marketplace update personal-skills
```

Or enable auto-update for the `personal-skills` marketplace in the `/plugin` menu —
then a push to this repo propagates to everyone on their next startup.

## Plugins

| Plugin | What it does | Invoke |
|---|---|---|
| `branch-merge-gro` | Merge branches into the shared `demo/gro` staging branch, in place, with a fixed safe routine (no new branches, no force-push, stop-and-ask on conflicts). | `/branch-merge-gro` |
| `pr-review-slack` | Turn GitHub PR links into a ready-to-send Slack review-request message (resolve titles, tag repos, pick reviewers, confirm, post). | `/pr-review-slack` |

## Layout

```
.claude-plugin/marketplace.json      # catalog listing the plugins
plugins/
├── branch-merge-gro/
│   ├── .claude-plugin/plugin.json
│   └── skills/branch-merge-gro/SKILL.md
└── pr-review-slack/
    ├── .claude-plugin/plugin.json
    └── skills/pr-review-slack/SKILL.md
```

To add a new skill: create `plugins/<name>/.claude-plugin/plugin.json` +
`plugins/<name>/skills/<name>/SKILL.md`, then add an entry to
`.claude-plugin/marketplace.json`.
