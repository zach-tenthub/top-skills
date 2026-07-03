---
name: top-deploy-prod
description: >
  Guide the HK team through a production deployment and help Tent of Presence engineers
  understand what they can support. Use this whenever anyone asks about deploying to production,
  releasing to master, scheduling a production release, or coordinating around go-live. Also
  trigger when someone asks "can I deploy to production?", "how does production deployment work?",
  "is it safe to push to master?", or "who approves a production release?". This skill makes
  roles and responsibilities crystal clear so no unscheduled production push happens.
---

# top-deploy-prod — Production Deployment Guide

Production deployments are the **HK team's responsibility**. Tent of Presence engineers support
but do not execute production releases and have no access to the production AWS environment.

If a Tent of Presence engineer asks how to deploy to production themselves — they cannot, and
should not. Redirect to the HK Tech Lead.

---

## Who does what

| Step | Responsible party |
|------|-------------------|
| Confirm staging is stable | HK Tech Lead |
| Schedule and announce the release | HK Tech Lead |
| Merge feature branch into `master` | HK engineer |
| Monitor GitHub Actions deploy | HK engineer |
| Watch `#deployment` for result | Both teams |
| Post-deploy verification on production | HK engineer |
| Support / answer questions during release | Tent of Presence (on request) |

---

## Production deployment process

### Stage 1 — Pre-release confirmation (HK Tech Lead)

Before any production push, the HK Tech Lead must confirm:
1. **Staging is stable.** The intended changes have been running on UAT without issues.
2. **The release is scheduled.** Production deployments are never ad hoc.

No production deployment happens without explicit HK Tech Lead sign-off — even for small
changes. Unscheduled pushes are hard to diagnose if something goes wrong outside a known
release window.

### Stage 2 — Merge into `master` (HK engineer)

Once sign-off is given, an HK engineer merges the feature branch into `master` via PR.
Use a **merge commit** (not squash) — production history must be complete for rollback
and incident investigation.

### Stage 3 — GitHub Actions deploys automatically

Merging into `master` triggers the production CI/CD pipeline. Both teams watch `#deployment`
for the result. If the pipeline fails, the HK Tech Lead decides on next steps.

### Stage 4 — Post-deploy verification (HK team)

HK team verifies the release on production. Tent of Presence engineers remain available
on Mattermost in case questions about the feature code come up.

---

## Hard rules

**No unscheduled production pushes.** In a genuine emergency, contact the HK Tech Lead first,
get verbal approval, then proceed.

**No production deployments on public holidays without prior agreement.** Both teams have
different holiday calendars (Malaysia and Hong Kong). Default position is to reschedule.

**Tent of Presence has no production AWS access.** If a production issue requires AWS
investigation, escalate to the HK team — do not attempt to access production infrastructure.

---

## What Tent of Presence engineers can do during a release

- Be available in Mattermost during the release window
- Answer questions about feature code from the HK team
- Watch `#deployment` alongside the HK team
- Verify the feature on staging beforehand (before HK Tech Lead sign-off)
- Escalate stability concerns to the HK Tech Lead *before* the release is scheduled

---

## Quick reference

```
HK Tech Lead confirms staging stable + schedules release
  → HK engineer merges feature branch into master via PR
    → GitHub Actions deploys to production automatically
      → Both teams watch #deployment
        → HK team verifies production
          → Tent of Presence available for support
```
