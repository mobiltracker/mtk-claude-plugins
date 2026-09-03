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

Follow the project's own conventions and checks — its CLAUDE.md, build/lint/test setup.
This skill adds the standards-fix loop on top; it doesn't override house rules.

## 0. Get the findings

- If a `review-standards` report for this scope already exists in the conversation, use it
  as the starting list.
- Otherwise, stop and tell the user to run `review-standards` for this scope first — do
  not run the review yourself, and do not fix blind.

## 1. Build the checklist

Format — numbered items, GitHub-style checkbox sub-steps:

```
**N. <short title>** (`file:line`)
- [ ] <sub-step>
- [ ] <sub-step>
```

- One numbered item per fix; the title carries `(file:line)`. Sub-steps are `- [ ]`
  checkboxes — the concrete edits/checks for that fix.
- Track state on the boxes: tick `- [x]` when done, with a short inline note
  (`— tested` · `— committed <sha>` · `— uncommitted` · `(extra)` for an emergent cleanup
  like a rename or reorder).
- **Separate standard-violations, incidental bugs, and product decisions.** State the
  split in a line (e.g. "itens 2-3 = bugs, não standards"); for standard items, name the
  `slug / rule-header`. Park product decisions — don't fix them as if they were standard
  violations.
- **Sketch a provisional commit plan** alongside the checklist — which items likely group
  into which commits (e.g. "utils · validation items · S/N"). Mark it a hypothesis: it
  gets adjusted as the code reveals what actually separates cleanly.
- Show the checklist. Get approval before editing anything — this skill mutates code.

## 2. Fix one item at a time

If you're on the default branch, create a feature branch **before the first commit** —
commits start firing inside this loop (see "Commit when a piece is closed"), so branch now.

Work each checklist item in order. For each:

- **Re-verify against the current file** — read the cited line, confirm the violation is
  still there. It may have moved (an earlier fix shifted lines) or already be resolved →
  if gone, mark obsolete, skip, tell the user.
- **Fix it where it lives.** If the fix belongs to shared code — a design-system
  component, a library, another module (a prop the atom doesn't forward, a modal missing
  an Esc handler) — don't force it into the target file. Mark the item out-of-scope, name
  the real owner, and suggest a separate issue instead.
- **A product decision is not a violation.** If an item turns out to hinge on a
  business/product choice the standard doesn't dictate (is this field required? is empty
  allowed?), don't fix it under the standards banner. Park it, surface the decision to the
  dev, and apply their call only if they make one.
- **Flag coupling/risk** before editing — JS depending on an `id`, a shared component, a
  value read elsewhere. Say what could break.
- **Propose the minimal isolated edit**, tied to the rule. Apply only on the user's
  go-ahead.
- **Stay within the rule.** The standard is the spec; if it names a reference
  implementation, that's the shape to match. When a fix needs a mechanism the standard
  does NOT specify (a cap scheme, a safety ceiling, a validator design), label it as your
  opinion — not the rule — and flag it before applying, so the dev decides. Same
  fact/opinion split review-standards uses for violação vs sugestão.
- **A port stays faithful to its source.** When you reuse a standard's reference
  implementation, don't fold another standard's rule into it. Keep the port pure and
  compose extra rules in a separate layer — e.g. emoji-blocking is an endereço rule, not
  text-input, so it belongs in an address helper, not the generic text util.
- **Say how to test it** — the concrete action (what to type / click / inspect). Offer to
  help if the dev needs it to run or reach the screen. Then wait for the user's result
  before the next item.
- **Commit when a piece is closed.** Once an item (or a small coherent group) is applied
  and tested green, commit it — reconcile against the provisional plan: if the code
  separated as planned, commit that boundary; if it interleaved, adjust the plan out loud
  and commit the unit that actually stands alone. Show the message, get approval.

Let cleanups surface as their own items (a rename, a DOM reorder) — don't force-bundle.

## 3. Commits

Driven by the provisional commit plan from §1, refined at each boundary as code lands.
Commit incrementally through the loop — each commit is the working-tree delta since the
last, so it's inherently one unit (no `git add -p` gymnastics). The plan is a guide, not a
promise: when reality diverges, say so and re-plan.

- Group into one commit only what doesn't stand alone (a new util + its first caller).
  **Never mix a standard-fix with an incidental bug.**
- Branch-first (done before the first commit, per §2). Show the message, get approval —
  never auto-commit.

## 4. Track state

Re-show the checklist whenever asked, in the §1 format — same numbered items and `- [ ]`
boxes, ticked `- [x]` with the inline note (tested · committed `<sha>` · uncommitted ·
`(extra)`). Obsolete items stay listed, struck as skipped. The developer always sees
what's done and what's left at a glance.

## 5. Wrap up

When every item is done, committed, or consciously skipped, offer to push the branch and
open a PR. Keep the offer generic (`git push` + `gh pr create`); don't run it without the
developer's go-ahead. Recap what shipped and what was deferred (out-of-scope / product
decisions).
