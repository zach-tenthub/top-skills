---
name: top-review
description: >
  Tent of Presence code review guide for HK engineers — use this whenever an HK engineer needs
  to review a PR from a Tent of Presence engineer, write review comments, check review SLAs,
  or decide whether a secondary review is needed. Trigger when someone asks "how do I review
  this PR", "what should I check", "is this blocking or a suggestion", "how long do I have to
  review this", or "do I need someone else to look at this". Also trigger when an author is
  following up on an overdue review and wants to know the escalation path.
---

# top-review — Tent of Presence Code Review Guide

All PRs from Tent of Presence require at least one HK engineer approval before merging.
Self-merge is not permitted.

## Who reviews

HK engineers review and approve all PRs from Tent of Presence engineers.

Request a **secondary review** from a fellow HK engineer for high-risk changes:
- Database migrations
- Authentication or authorisation changes
- Infrastructure changes (CloudFormation, IAM, CI/CD)

## What to check

Work through these areas in order — correctness first, then security, then the rest.

### Correctness
Read the linked GitHub Issue and acceptance criteria before the diff. A clean PR that solves
the wrong problem still needs to go back.

### Security — OWASP Top 10 awareness

| Risk | What to look for |
|------|------------------|
| Injection | Raw SQL strings; unsanitised input in Node/PHP; use parameterised queries |
| Broken authentication | Insecure JWT; missing token expiry; weak session management |
| Sensitive data exposure | PII or payment data in logs; unencrypted data at rest/transit |
| Broken access control | Missing auth checks on API endpoints (IDOR patterns) |
| Security misconfiguration | Debug mode on; open S3 buckets; default credentials |
| Insecure deserialisation | Untrusted JSON parsed without validation in Node.js or PHP |
| Outdated components | New packages with known vulnerabilities (`npm audit`) |

### Performance
- N+1 queries: a query inside a loop that should be batched
- Missing indexes on columns in WHERE, JOIN, or ORDER BY
- Unoptimised loops over large datasets that could be handled at DB level

### Readability
Is the code self-explanatory? Variable and function names should describe intent, not implementation.

### Test coverage
New features and bug fixes should include tests, especially for edge cases from the ticket.

### Conventions
Does it follow project conventions — file structure, naming patterns, formatting?

## Comment prefixes

Every review comment must start with one of these so the author knows how to triage:

| Prefix | Meaning |
|--------|---------|
| `[blocking]` | Must be addressed before the PR can be merged |
| `[suggestion]` | Non-blocking; author may choose to act or defer |
| `[question]` | Seeking clarification; not necessarily a problem |

Authors must respond to every comment — with a code change, explanation, or a deferred ticket.

**Write effective comments** — suggest an alternative, don't just flag a problem:

```
[blocking] The query on line 42 fires once per user inside the loop — N+1 issue.
Move it above the loop and use an IN clause:
  SELECT * FROM orders WHERE user_id IN (...)
Then map results by user_id before iterating.

[suggestion] `getUserData` could be renamed `fetchUserById` — clearer it hits the DB.

[question] Is there a reason we're not using `authMiddleware` here? Wondering if intentional.
```

## Review SLA

| Situation | Target turnaround |
|-----------|-------------------|
| PR tagged `priority: urgent` | Same business day |
| Standard PR | 1 business day |
| Large PR (>200 lines of logic) | Up to 2 business days |

If a PR breaches SLA without a response, the author should follow up in `#engineering`
Mattermost with a direct `@mention` of the reviewer. This is the documented escalation path.

## Review checklist

- [ ] Read the linked ticket and acceptance criteria before the diff
- [ ] Checked correctness against ticket requirements
- [ ] Scanned for OWASP Top 10 risks
- [ ] Checked for N+1 queries, missing indexes, performance issues
- [ ] Assessed readability — variable names, function names, code flow
- [ ] Verified test coverage for new and changed logic
- [ ] All comments use `[blocking]`, `[suggestion]`, or `[question]` prefix
- [ ] Considered whether secondary review is needed (DB migrations, auth, infra)
