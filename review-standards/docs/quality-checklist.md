# Quality Checklist — review-standards plugin

Rubric for judging a run of either skill in this plugin. Two skills, two sections —
apply the one that matches what ran. Each box is a pass criterion; the italic note names
the failure it guards against.

The plugin has two skills:
- **review-standards** — read-only; reports violations. Never edits.
- **fix-standards** — mutating; turns a review report into applied, committed fixes.

---

## review-standards (read-only report)

### Scope & entry
- [ ] No-arg → asks for scope (path/branch/PR/repo); never auto-launches a whole-repo audit _(whole-repo default was too aggressive)_
- [ ] Scope resolved fast — recognizes path/branch/PR, or asks once
- [ ] KB located via `KNOWLEDGE_BASE_DIR` (or default); if missing, tells the user to clone or set the env var
- [ ] Reads every applicable standard before judging

### Findings & classification
- [ ] Reads the actual code — validator logic, JSX, attrs — not filename guesses
- [ ] Catches violation-by-absence (a required behavior simply missing)
- [ ] Separates standard-violations, incidental bugs, and product decisions
- [ ] Component-dependent findings (behavior lives in a shared component) → `sugestão` to verify, not a flat `violação`; offers to confirm by reading the component

### Output
- [ ] Grouped file → line, nested, real checkboxes where applicable
- [ ] `violação` = fact (code diverges from a named rule); `sugestão` = opinion (fix)
- [ ] References the rule as `slug / rule-header`; no verbose verbatim quoting by default
- [ ] No severity labels (unsourced); clean file → `conforms`, no padding
- [ ] Ends by pointing to `fix-standards` when there are findings

### DevEx
- [ ] Invents no work — 0 relevant → says so
- [ ] Low ceremony; no tool-flail while waiting on subagents

---

## fix-standards (guided fix loop)

### Entry
- [ ] No blind fix — no report in context → tells user to run review first
- [ ] Branch-first BEFORE the first commit (don't commit on the default branch then fix up)

### Checklist (run-up) — where ceremony creeps in
- [ ] Checklist is lean — titles + `file:line`, no piled-up coupling/risk/open-questions prose
- [ ] Rendered as real checkboxes (`- [ ]` / `- [x]`), not plain `-` bullets
- [ ] Titles, not concrete code — code for an item is written only when reached
- [ ] Works top-to-bottom — no A/B/C menu or ordering ceremony
- [ ] Standard vs bug vs product-decision split stated in one line
- [ ] Provisional commit plan sketched as a hypothesis
- [ ] Few turns to first edit — run-up isn't a decision ping-pong

### Fix design (non-trivial items)
- [ ] Reuse-first for real — searches `utils/` + schemas + the standard's ref BEFORE designing new code
- [ ] Design anchored once — no re-deriving or re-proposing alternatives across turns
- [ ] Faithful to ref — ports the reference impl; doesn't fold another standard's rule in
- [ ] Opinion labeled proactively — a mechanism outside the standard (maxLength, cap, infra) flagged as opinion before the dev asks

### Edit loop
- [ ] Binding — one piece per cycle; a single "y" never approves a batch
- [ ] Re-verify before each edit — current line, violation still live
- [ ] Minimal isolated edit — no infra beyond what the fix needs
- [ ] Test and wait for the dev's result before advancing

### Commits / branch
- [ ] Commit-gate — closed piece committed (or explicitly deferred) before the next edit
- [ ] No `git add -p` / hunk gymnastics — clean incremental commits
- [ ] Standard-fix never mixed with an incidental bug in one commit

### Fidelity / honesty
- [ ] Out-of-scope handled — lib/design-system fix → separate issue, not forced into the file
- [ ] Product decision returned to the dev — not fixed under the standards banner
- [ ] Honest self-correction — when wrong (DB column, wrong finding), admits and re-checks

### Wrap-up & DevEx
- [ ] Offers push/PR generically at the end; recaps shipped vs deferred
- [ ] Low ceremony — energy on the fix, not on checklist/branch/commit-msg ritual
