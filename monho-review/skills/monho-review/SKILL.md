---
name: monho-review
description: >-
  Run the monho code review locally and print findings to the console. Use when asked
  to review the current branch or a PR for bugs, security, performance, and
  encapsulation, the same review @monho-review posts on GitHub. Nothing is posted.
allowed-tools: Read, Grep, Glob, Bash(gh api:*), Bash(gh pr diff:*), Bash(gh pr view:*), Bash(git diff:*), Bash(git merge-base:*), Bash(git remote show:*), Bash(git remote get-url:*)
---

# Monho Review

Runs the same review as the `@monho-review` GitHub Action, in the console. The prompt is
fetched from the action at run time, so this skill follows any change to it.

## 1. Target

- No argument: the current branch diff against the default branch. Resolve the default
  branch with `git remote show origin` (HEAD branch), then use
  `git diff $(git merge-base origin/<default> HEAD)...HEAD`.
- `--pr <n>`: pull request `<n>` of the current repo. Use `gh pr diff <n>`.

## 2. Fetch the prompt

Run:

```
gh api repos/mobiltracker/gh-actions/contents/gh/monho-review/action.yaml -H "Accept: application/vnd.github.raw"
```

If it fails, STOP and tell the user:
`monho-review precisa do gh (GitHub CLI) instalado e logado com acesso à org mobiltracker. Instale em https://cli.github.com e rode gh auth login.`

From the file, take the `prompt` of the step that uses `anthropics/claude-code-action`.
Replace `${{ github.repository }}` with the current repo, derived from
`git remote get-url origin`, and `${{ inputs.pr_number }}` with `<n>`, or with
`local branch` when there is no PR.

## 3. Run the review

Follow the fetched prompt, with these local overrides:

- Without `--pr`, read the diff from §1 instead of `gh pr diff`.
- Do not post anything. Ignore every instruction to comment on the PR.
- Print each inline comment to the console, prefixed with `file:line`, keeping its
  Problema/Cenário/Sugestão format.
- Print the summary paragraph at the end.
