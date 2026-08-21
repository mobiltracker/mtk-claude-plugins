---
name: update-mtk-plugins
description: >-
  Use when the user asks to update, sync, or refresh MTK plugins/skills from the
  mtk-claude-plugins marketplace — e.g. a teammate pushed a new or changed skill and the
  user wants it locally, or the user just says "atualiza os plugins" / "puxa a skill
  nova".
---

# Update MTK Plugins

Pulls the latest marketplace metadata and updates every installed plugin from the
`mtk-claude-plugins` marketplace, then tells the user the one step this skill cannot do
for them.

## 1. Update the marketplace

```
claude plugin marketplace update mtk-claude-plugins
```

If this marketplace name isn't configured, run `claude plugin marketplace list` to find
the right name and use that instead.

## 2. Update every plugin from it

```
claude plugin list --json
```

Filter to plugins whose marketplace is `mtk-claude-plugins`, then for each one:

```
claude plugin update <plugin-name>@mtk-claude-plugins
```

Report any failures per plugin instead of stopping the whole run.

## 3. Tell the user to reload

Plugin updates apply on disk but do not take effect in the running session. There is no
tool or CLI command that reloads the *current* session's plugins — that is a REPL-only
action. End by telling the user, verbatim:

> Atualizado. Roda `/reload-plugins` pra aplicar nessa sessão.

Do not claim the update is "already active" — it isn't until the user runs that command.
