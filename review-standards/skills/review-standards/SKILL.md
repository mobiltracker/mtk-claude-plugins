---
name: review-standards
description: >-
  Review frontend code against Mobiltracker knowledge-base standards (BR person-name
  validation, BR address forms, and any future docs/standards/*.md). Use when asked to
  check, audit, or review code against the standards — a form, a file/dir, a PR, a
  commit, or a branch diff — for standards compliance. Data-driven: automatically picks
  up new standards added to the knowledge-base with no skill changes.
---

# Review Standards

Review frontend code against the reusable standards in the local knowledge-base clone.
Read-only: report violations, do not auto-fix. The developer decides what to change.

## 0. Locate the standards

Standards live in the `knowledge-base` clone, at `docs/standards/`. Resolve its location as
`${KNOWLEDGE_BASE_DIR:-~/git/knowledge-base}` — honor the `KNOWLEDGE_BASE_DIR` env var if
set, otherwise default to `~/git/knowledge-base` (some devs clone elsewhere, e.g.
`~/Projetos`).

- If the resolved directory does not exist, STOP and tell the user:
  `knowledge-base not found at <resolved path>. Clone it (git clone <repo url> ~/git/knowledge-base) or, if cloned elsewhere, set KNOWLEDGE_BASE_DIR in ~/.claude/settings.json (env) to its path.`
  Do not attempt to review without it.
- The knowledge-base lives outside the project workspace, so reading it prompts for folder
  access on every run. To silence that once, suggest the user add its path to
  `permissions.additionalDirectories` in `~/.claude/settings.json`. Suggest only — do not
  edit their settings yourself.
- List every `*.md` **standard document** in that directory. A standard document is a
  top-level `*.md` describing a rule (e.g. `br-person-name-validation.md`,
  `endereco-formulario-br.md`). Skip `README.md` and any index/meta file — they describe
  the collection, not a rule. Some standards have a sibling folder with a test harness
  (`cases.json` / `ref.mjs` / `runner.mjs`); note it but do not run it (this skill is
  LLM-review only).
- Read each standard document in full. These are the rules of record — reference them by
  `slug / rule-header` when reporting, and quote wording only when it clarifies a subtle
  divergence (per §5). Standards are written in Portuguese; rules apply regardless.

This step is dynamic on purpose: whatever `*.md` files exist at runtime are the standards.
Adding a new standard to the knowledge-base needs no change to this skill.

## 1. Resolve scope

Parse the invocation argument to decide which files to review.

| Invocation | Scope | Narrowing |
|---|---|---|
| `/review-standards` (no arg) | ask the user for scope | route per their choice |
| `/review-standards --branch` | `git diff --name-only main...HEAD` | **C — none** (§3) |
| `/review-standards --commit <sha>` | `git diff --name-only <sha>~1 <sha>` | **C — none** (§3) |
| `/review-standards --pr <n>` | `gh pr diff <n> --name-only` (needs `gh`) | **C — none** (§3) |
| `/review-standards <path>` | that file or dir (recursively) | **C — none** (§3) |

Notes:
- No arg → **ask what to review before doing anything**: a path, `--branch`, `--pr <n>`,
  or the whole repo. Whole-repo audit (mode B, §2) is the most expensive and rarely the
  intent — run it only if the user explicitly picks it. Never auto-launch it.
- `main` is the base branch; if the repo's default branch differs (e.g. `master`),
  detect it (`git symbolic-ref refs/remotes/origin/HEAD`) and use that.
- For `--pr`, if `gh` is unavailable, tell the user and fall back to `--branch`.
- Restrict to source files (`.ts`, `.tsx`, `.js`, `.jsx`, `.cs`, `.razor`, etc.).
  Ignore `node_modules`, `dist`, `build`, generated files, lockfiles.

## 2. Whole-repo narrowing — semantic scan (mode B)

Do NOT grep for keywords. Keyword grep misses **violation-by-absence** (e.g. an address
form that has *no* CEP lookup at all is exactly what the address standard should flag,
but it never mentions "cep"). Instead:

- Launch an **Explore subagent** (read-only) with a brief derived from the standards you
  read in §0. For each standard, tell the subagent what a governed file looks like AND
  what a file that *should* comply looks like. Example brief:
  > Find files that are, or should be, (a) person-name input/validation — any form field
  > or validator handling a person's name (nome/prenome/sobrenome); (b) BR address entry —
  > any form capturing endereço/CEP/cidade/UF, or address validation/lookup code. Include
  > files that handle these concepts even if they don't use those exact words. Return
  > file paths grouped by which concept they match.
- The subagent returns candidate files per standard. Those are the review set.
- If the scan returns nothing for any standard, report "0 relevant files found for
  <standard>" and move on — do not invent work.

## 3. Changed-scope — no narrowing (mode C)

The file set from §1 is already small. Review **all** of it directly against every
standard. Skip the Explore subagent. A changed file is in scope for a standard if the
standard plausibly governs it; skip files no standard applies to and list them in the
mode-C coverage line (per §5).

## 4. Review

For each in-scope file, against each applicable standard:

- Check **every** rule in the standard document — both the mechanically-verifiable rules
  (e.g. name length min/max, allowed/forbidden charset, forbidden combinations, RN055
  term list; CEP mask, 8-digit strip, UF closed-list) AND the UX/semantic rules
  (e.g. lookup on blur not keystroke, don't lock fields readonly, reset dependents when
  CEP cleared, textual error message not color-only, validate on blur/submit).
- Read the actual code — validator logic, JSX, event handlers, input attributes — to
  judge conformance. Don't assume from filenames.
- Watch for violation-by-absence: a required behavior that is simply missing.
- When a rule's conformance depends on a shared component the file imports (validation
  timing, prop forwarding, Esc handling) rather than code visible here, you can't confirm
  it from this file alone. Don't assert a flat `violação` — record it as a `sugestão` to
  verify, naming the component and what to check. (You'll offer to confirm these in §5.)

## 5. Report

Grouped by **file → line**. Only files/lines with at least one violation appear —
no block for a clean file.

```
<file>
  :<line> — <what the element/field is>
    violação  <standard-slug> / <rule-header> — <what in the code diverges>
    violação  <standard-slug> / <rule-header> — <...>
    sugestão  <how to fix>
  :<line> — <...>
    violação  <...>
    sugestão  <...>

<other-file>
  :<line> — <...>
    ...
```

Three levels: **file** → **`:line` + what it is** → **violação / sugestão**. The file
appears once; its lines nest under it.

Two layers of content, kept distinct on purpose:

- **`violação`** — a *fact*: the code diverges from a rule of record. Always name the
  rule as `<standard-slug> / <rule-header>` (the doc's own section header) so the dev can
  open the source and verify. One `violação` line per violated rule.
- **`sugestão`** — the skill's *opinion* on how to fix it. At most one per line; when
  several violations hit the same spot, consolidate into a single fix. The standard may
  give a direction (quote it if so), but the concrete steps are a suggestion — the
  `sugestão` label marks them as opinion the dev takes or ignores.

Rules:

- **No severity.** Do not label findings blocker/warn/note. The standards carry no
  severity, so any level would be reviewer opinion dressed as data. Report the violation;
  the dev prioritizes.
- **No inline verbatim quoting** by default — `<standard-slug> / <rule-header>` already
  points to the exact rule. Quote the doc only if the divergence is subtle and the wording
  clarifies it.
- Order by file, then line.
- **Coverage (mode C only):** after the findings, add one line
  `pulados: <files> (nenhum padrão aplica)` listing files that were in scope but skipped
  because no standard governs them. This is per-file coverage, distinct from the
  per-standard N/A forbidden below.
- If the whole scope is clean against every applicable standard, say `conforms` — don't
  pad. Do not list standards that didn't apply.
- If any findings are component-dependent and unverified (§4), close by offering to confirm
  them: read the components the file imports and reclassify — confirmed defect → `violação`
  (name the owner); not a defect → drop. Read only on the user's go-ahead; the default
  review stays light.
- When the report has findings, close with a one-line pointer: to apply them, run
  `fix-standards` for this scope — it turns this report into a guided fix checklist. Skip
  the pointer when everything conforms.

Do not edit any file. This skill only reports.
