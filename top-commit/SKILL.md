---
name: top-commit
description: >
  Tent of Presence commit message guide — use this whenever you need to write a commit message,
  check if a commit message is formatted correctly, or pick the right commit type. Trigger when
  someone asks "how should I write this commit", "what type should I use", "is this commit message
  correct", or pastes a diff and wants a properly formatted message. Also trigger when someone
  is about to push and wants to verify their commits follow Conventional Commits before the PR
  is opened.
---

# top-commit — Tent of Presence Commit Message Guide

This skill encodes the Conventional Commits standard used across all Tent of Presence repositories.
Consistent commit messages make the changelog readable, help reviewers understand scope, and let
automated tools parse history reliably.

## Format

```
<type>(<scope>): <short summary>

[optional body]

[optional footer: refs #ticket-id]
```

Every commit message must be in **English** — this keeps the audit trail readable for both the
Malaysia and HK teams regardless of meeting language.

## Types

Pick the type that matches the **intent** of the change, not the files touched.

| Type | Use when |
|------|----------|
| `feat` | Adding new functionality visible to users or API consumers |
| `fix` | Correcting a bug — something was broken and now it works |
| `chore` | Maintenance that doesn't change behaviour: deps, config, scripts |
| `refactor` | Code restructured with no functional change — same inputs, same outputs |
| `test` | Adding or updating tests only |
| `docs` | Documentation only — READMEs, inline comments, guides |
| `ci` | Changes to GitHub Actions workflows or CI scripts |

When in doubt: if a user would notice the change → `feat` or `fix`. If only a developer
would notice → `refactor`, `chore`, or `test`.

## Scope

Optional but recommended. Names the part of the system affected.

Common scopes: `auth`, `api`, `db`, `frontend`, `mobile`, `deps`, `infra`, `ci`

## Short summary

- Keep under **72 characters** (fits in terminals without wrapping)
- Use **imperative mood**: "add", "fix", "remove" — not "added", "fixes", "removing"
- Do not end with a period

The imperative mood matches how git itself phrases things ("Merge pull request…") so history
reads consistently.

## Body

Add a body when the summary alone won't tell a future reader *why* you made the change.
Separate from the summary with a blank line. Wrap at 72 characters.

## Footer

Link to tickets so the GitHub Projects board stays traceable:
```
refs #123
```

## Examples

**Feature with scope and ticket:**
```
feat(auth): add JWT refresh token rotation

Rotates the refresh token on each use to mitigate token theft.
Single-use tokens stored in Redis with a 7-day TTL.

refs #proj-89
```

**Bug fix:**
```
fix(api): handle null response from payment gateway

Stripe webhook occasionally returns null for payment_intent on
card declines before authorization. Guard added at response parsing.

refs #proj-112
```

**Dependency update:**
```
chore(deps): update express to v4.19.0
```

**Tests only:**
```
test(auth): add unit tests for token expiry edge cases
```

## Commit message checklist

- [ ] Type is one of: feat, fix, chore, refactor, test, docs, ci
- [ ] Summary is in English, imperative mood, under 72 characters, no trailing period
- [ ] Scope included if the change is scoped to a subsystem
- [ ] Body explains the *why* if the summary is not self-evident
- [ ] Footer includes `refs #ticket-id` if there is a linked ticket
