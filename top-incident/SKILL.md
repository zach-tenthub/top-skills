---
name: top-incident
description: >
  Incident reporting guide for Tent of Presence engineers. Use this the moment something goes
  wrong in any environment — production errors, staging breakages, unexpected data, service
  outages, or security alerts. Trigger when you're about to post in #incidents, unsure whether
  something is worth reporting, or a teammate asks "should we raise an incident?" — the answer
  is almost always yes. This skill tells you exactly what severity to assign, what to post,
  and who to notify.
---

# top-incident — Incident Reporting Guide

When something breaks, resist the instinct to fix first and tell someone later. Early
notification lets the right people make informed decisions, creates a record if the issue
escalates, and means no one is surprised.

---

## Step 1 — Assess Severity

| Level | Name | What it means |
|-------|------|---------------|
| **P1** | Critical | Production down, data at risk, or security breach suspected. Zero workaround. |
| **P2** | High | Significant production degradation — key feature broken, no workaround for users. |
| **P3** | Medium | Non-critical; a workaround exists and users can continue. |
| **P4** | Low | Cosmetic issue; no functional impact. |

When in doubt between two levels, pick the higher one. You can downgrade later.

---

## Step 2 — Know Your Role

**Production incidents:** Tent of Presence engineers do not attempt to fix production.
The HK team owns production. Your job: observe accurately, report promptly, support with
information. Attempting production fixes independently risks worsening the incident and
creates confusion about system state.

**Staging incidents:** You may attempt a fix independently. Still notify `#incidents`
so there is a record and others don't investigate the same issue.

---

## Step 3 — Post in #incidents immediately

Post before you have a full picture. An incomplete post now is more valuable than a
complete post in 30 minutes. Update the thread as you learn more.

```
**[P?] [Short title]**

**Observed:** What you saw — error message, wrong behaviour, alert fired.
**Started:** When first noticed (approximate is fine).
**Environment:** production / staging / dev
**Affected:** Which service, endpoint, page, or user group.
**Tried so far:** Any steps already taken (even "checked logs, no obvious cause").
```

**Example:**
```
**[P2] Checkout API returning 500 for all users**

Observed: All POST /api/checkout returning 500. Sentry: TypeError in payment-service.js:84.
Started: ~14:32 HKT.
Environment: production
Affected: All users attempting checkout.
Tried so far: Recent deploy at 14:20 HKT may be related.
```

---

## Step 4 — Tag the Right People

- **P1 and P2:** Tag `@matt` (HK Tech Lead) directly in the `#incidents` post. Do not wait
  for him to notice the channel.
- **P3 and P4:** Post in `#incidents` without a direct tag.

---

## Step 5 — Outside Working Hours

Post in `#incidents` as normal. No immediate response expected — the post creates a record
for the morning. For P1, note clearly in the post that it cannot wait.

---

## Step 6 — Keep the Thread Updated

Add updates to the same thread. Update when severity changes, cause is identified, fix
is deployed, or the incident resolves.

**Resolution update:**
```
**Resolved** [HH:MM HKT]
Resolution: [what was done / what was reverted]
Duration: approximately [X] minutes
Next step: post-mortem will be initiated by HK Tech Lead (P1/P2 only)
```

---

## Quick Reference

| Scenario | Severity | Tag @matt? | May attempt fix? |
|----------|----------|------------|-----------------|
| Production down | P1 | Yes | No |
| Production data at risk | P1 | Yes | No |
| Production feature broken, no workaround | P2 | Yes | No |
| Production degraded, workaround exists | P3 | No | No |
| Staging broken | any | No | Yes |
| Cosmetic bug | P4 | No | Staging: yes |

---

## What Not to Do

- Don't wait until you understand the cause before posting.
- Don't attempt to fix production to avoid raising an incident.
- Don't DM @matt without also posting in `#incidents` — the channel is the record.
- Don't close the thread without a resolution update.
