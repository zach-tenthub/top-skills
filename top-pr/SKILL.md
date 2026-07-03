---
name: top-pr
description: >
  Guide engineers at Tent of Presence Sdn Bhd through creating a pull request
  that fully complies with the team's working guide. Use this skill whenever
  the user mentions opening a PR, creating a pull request, submitting changes
  for review, or asking how to push their branch. Also trigger when the user
  says things like "I'm done with my feature", "ready to merge", "how do I
  submit this", or "can you help me write my PR description". This skill
  enforces branch naming, commit message convention (Conventional Commits),
  the pre-flight checklist, and the required PR description template.
---

# top-pr — Tent of Presence PR Standard

You are helping an engineer open a pull request that follows Tent of Presence's
working guide. Your job is to guide them through every required step so the PR
is complete, correct, and ready for the HK team to review without back-and-forth.

---

## Step 1 — Confirm the branch name

Check the current branch with `git branch --show-current`.

Branch names must follow this pattern exactly:

| Work type      | Pattern                              | Example                          |
|----------------|--------------------------------------|----------------------------------|
| New feature    | `feature/TICKET-ID-short-description`| `feature/proj-42-job-search-api` |
| Bug fix        | `fix/TICKET-ID-short-description`    | `fix/proj-87-csrf-token-null`    |
| Maintenance    | `chore/short-description`            | `chore/upgrade-sequelize-v7`     |

Rules:
- Always **lowercase and hyphenated** — no spaces, no camelCase, no underscores
- Must have been **created from `master`**, not from `uat` or any other branch
- Never commit directly to `master`, `uat`, or `release/*`

If the branch name is wrong, show the rename command:
```bash
git branch -m <old-name> <new-name>
```

---

## Step 2 — Rebase on latest master

```bash
git fetch origin
git rebase origin/master
```

If there are conflicts, help the engineer resolve them file by file, then `git rebase --continue`.

---

## Step 3 — Run the pre-flight checklist

- [ ] **Self-review**: Read the diff (`git diff origin/master`) as if you were the reviewer.
- [ ] **Tests pass**: All existing tests pass locally. New functionality has tests where applicable.
- [ ] **No secrets**: No hardcoded credentials, API keys, passwords, or tokens in the diff.
- [ ] **Branch is rebased**: Confirmed in Step 2.
- [ ] **Commit messages follow Conventional Commits**: See Step 4.

---

## Step 4 — Validate commit messages

```
<type>(<scope>): <short summary>

[optional body]

[optional footer: refs #ticket-id]
```

| Type | When to use |
|------|-------------|
| `feat` | A new feature |
| `fix` | A bug fix |
| `chore` | Maintenance, dependency updates, config |
| `refactor` | Code change with no functional difference |
| `test` | Adding or updating tests |
| `docs` | Documentation only |
| `ci` | CI/CD configuration changes |

Check commits: `git log origin/master..HEAD --oneline`

Fix non-compliant messages: `git rebase -i origin/master`

---

## Step 5 — Generate the PR description

```markdown
## Summary
<!-- What does this PR do? Why is it needed? -->

## Changes
<!-- Bullet list of notable changes -->
- 

## Testing
<!-- How was this tested? What edge cases were considered? -->

## Linked Ticket
Closes #
```

Ask the engineer to describe their changes in plain language, then draft the description.
Do not just copy commit messages — synthesise them into readable prose.

---

## Step 6 — Remind about review requirements

1. **All PRs from Tent of Presence must be reviewed and approved by at least one HK engineer.** Self-merge is not permitted.
2. Move the linked ticket to **"In Review"** and attach the PR link.
3. Standard review SLA: **1 business day**. Follow up in `#engineering` with `@mention` if breached.
4. Large PRs (>200 lines of logic): up to **2 business days**.
5. Tag `priority: urgent` for same business day turnaround.

---

## Full flow summary

```
1. Check branch name → rename if wrong
2. Rebase on origin/master
3. Pre-flight checklist (5 items)
4. Validate commit messages (Conventional Commits)
5. Write PR description (Summary / Changes / Testing / Linked Ticket)
6. Open PR → move ticket to "In Review" → wait for HK review
```
