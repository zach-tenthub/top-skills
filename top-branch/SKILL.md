---
name: top-branch
description: >
  Tent of Presence engineering branching guide — use this whenever you need to create a branch,
  name a branch, check if a branch name is valid, or understand the branching rules. Trigger
  whenever someone asks "what should I name my branch", "how do I create a branch", "can I branch
  from uat", "is my branch name correct", or anything involving git branching conventions.
  Also trigger when someone is about to open a PR and needs to verify their branch is rebased
  and named correctly.
---

# top-branch — Tent of Presence Branch Guide

This skill encodes the Tent of Presence branching rules so you never have to look them up mid-flow.

## Branch types and naming

| Type | Pattern | When to use |
|------|---------|-------------|
| Feature | `feature/TICKET-ID-short-description` | New functionality tied to a GitHub Issue |
| Fix | `fix/TICKET-ID-short-description` | Bug fixes tied to a GitHub Issue |
| Chore | `chore/short-description` | Maintenance, config, dependency updates |

Names must be **lowercase and hyphenated**. Spaces and camelCase break shell scripts and URL slugs.

**Good examples:**
```
feature/proj-123-user-auth
fix/proj-456-null-payment-response
chore/update-express-v5
```

**Bad examples (and why):**
```
feature/userAuth            ← camelCase
fix/Proj-456                ← uppercase
feature/proj-123_user_auth  ← underscores
```

## Where to branch from

Branch from `master` only — always.

`uat` is a deployment target, not a development base. Branching from `uat` inherits unreviewed
changes from other engineers. Branching from `master` guarantees a clean, production-equivalent base.

Never create branches from `uat` or `release/*`.

## Creating a branch

```bash
# Make sure master is current first
git checkout master
git pull origin master

# Create and switch to your branch
git checkout -b feature/proj-123-short-description
```

## Before opening a PR — rebase on master

Rebasing brings your branch up to date with changes that landed on `master` while you worked.
This prevents conflicts from falling on the reviewer instead of the author.

```bash
git fetch origin
git rebase origin/master
```

Then push with `--force-with-lease` (safer than `--force` — fails if someone else pushed to
your branch in the meantime):

```bash
git push origin feature/proj-123-short-description --force-with-lease
```

## Protected branches — never commit directly

- `master` — production code, HK team only
- `uat` — UAT environment, CI/CD managed
- `release/*` — release branches, HK team only

Always work in a feature, fix, or chore branch and go through the PR process.

## Keep branches short-lived

Merge or close branches within **2 weeks**. Long-lived branches accumulate drift from `master`,
making rebases painful and reviews harder. Break large features into smaller mergeable chunks.

## Branch checklist before opening a PR

- [ ] Branch created from `master` (not `uat`)
- [ ] Name matches `feature/`, `fix/`, or `chore/` — lowercase, hyphenated
- [ ] Branch is rebased on latest `master`
- [ ] Branch has been open for fewer than 2 weeks
