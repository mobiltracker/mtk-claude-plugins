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

Standards live at `~/git/knowledge-base/docs/standards/`.

- If that directory does not exist, STOP and tell the user:
  `knowledge-base not cloned. Run: git clone <knowledge-base repo url> ~/git/knowledge-base`
  Do not attempt to review without it.
- List every `*.md` **standard document** in that directory. A standard document is a
  top-level `*.md` describing a rule (e.g. `br-person-name-validation.md`,
  `endereco-formulario-br.md`). Skip `README.md` and any index/meta file — they describe
  the collection, not a rule. Some standards have a sibling folder with a test harness
  (`cases.json` / `ref.mjs` / `runner.mjs`); note it but do not run it (this skill is
  LLM-review only).
- Read each standard document in full. These are the rules of record — quote them
  exactly when reporting. Standards are written in Portuguese; rules apply regardless.

This step is dynamic on purpose: whatever `*.md` files exist at runtime are the standards.
Adding a new standard to the knowledge-base needs no change to this skill.

## 1. Resolve scope

Parse the invocation argument to decide which files to review.

| Invocation | Scope | Narrowing |
|---|---|---|
| `/review-standards` (no arg) | whole repo | **B — semantic scan** (§2) |
| `/review-standards --branch` | `git diff --name-only main...HEAD` | **C — none** (§3) |
| `/review-standards --commit <sha>` | `git diff --name-only <sha>~1 <sha>` | **C — none** (§3) |
| `/review-standards --pr <n>` | `gh pr diff <n> --name-only` (needs `gh`) | **C — none** (§3) |
| `/review-standards <path>` | that file or dir (recursively) | **C — none** (§3) |

Notes:
- Default (no arg) = **whole-repo audit**. Find all violations, not just new ones.
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
standard plausibly governs it; skip files no standard applies to and say so briefly.

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
- If a standard is pure process / not code-checkable, note it as skipped for that file.

## 5. Report

Output plain markdown, grouped by standard. For each finding:

```
<file>:<line> — [<standard-slug> / <rule>] <what's wrong> → <fix>
```

- Prefix each finding with severity: **blocker** (breaks the rule outright / bad data
  downstream), **warn** (deviates, lower risk), **note** (minor / stylistic).
- Order findings most-severe first within each standard.
- When a file/standard is clean, say `conforms` — don't pad.
- End with a one-line summary: files reviewed, findings by severity.
- Cite the standard's own wording for each violated rule so the dev can verify against
  the source doc.

Do not edit any file. This skill only reports.
