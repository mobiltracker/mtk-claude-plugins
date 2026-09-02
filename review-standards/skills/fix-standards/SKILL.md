---
name: fix-standards
description: >-
  Guided correction of standards violations found by review-standards. Turns a review
  report into a checklist and applies the fixes one at a time, with local testing between
  each and granular commits. Use after a review, when the user wants to actually fix the
  findings — e.g. "corrige isso", "vamos arrumar os findings", "aplica as correções".
  Edits code and commits: asks approval before each edit and before each commit.
---

# Fix Standards

Apply fixes for `review-standards` findings. Interactive and incremental — the developer
approves each edit and tests as they go. Companion to `review-standards`, which is
read-only; this skill is the one that mutates code.

## 0. Get the findings

- If a `review-standards` report for this scope already exists in the conversation, use it
  as the starting list.
- Otherwise, stop and tell the user to run `review-standards` for this scope first — do
  not run the review yourself, and do not fix blind.

## 1. Build the checklist

- One checklist item per fix; nest sub-steps under it.
- **Separate standard-violations from incidental bugs** you notice along the way. Mark
  which items come from a standard (name the `slug / rule-header`) and which are just bugs.
- Show the checklist. Get approval before editing anything — this skill mutates code.

## 2. Fix one item at a time

Work each checklist item in order. For each:

- **Re-verify against the current file** — read the cited line, confirm the violation is
  still there. It may have moved (an earlier fix shifted lines) or already be resolved →
  if gone, mark obsolete, skip, tell the user.
- **Flag coupling/risk** before editing — JS depending on an `id`, a shared component, a
  value read elsewhere. Say what could break.
- **Propose the minimal isolated edit**, tied to the rule. Apply only on the user's
  go-ahead.
- **Say how to test it**; wait for the user's result before the next item.

Let cleanups surface as their own items (a rename, a DOM reorder) — don't force-bundle.

## 3. Commit in isolated logical chunks

- Group related hunks into one commit; **never mix a standard-fix with an incidental bug**
  in the same commit.
- Show the proposed message and get approval before committing. Do not commit
  automatically.

## 4. Track state

Re-show the checklist whenever asked. Mark each item done / obsolete / pending so the
developer always knows what's left.
