---
name: top-postmortem
description: >
  Post-mortem writing guide for Tent of Presence P1 and P2 incidents. Use this when the HK
  Tech Lead has initiated a post-mortem (within 2 business days of a P1 or P2), when you're
  asked to contribute to or draft a post-mortem, or when a resolved critical incident needs a
  written record of what happened and how to prevent recurrence. Also trigger when someone asks
  "how do we write a post-mortem?" or "what goes in the post-mortem?".
---

# top-postmortem — Post-Mortem Guide

Post-mortems exist to make the system more reliable, not to establish who was at fault.
A blameless post-mortem produces better outcomes — people share what actually happened
rather than what looks defensible.

**When triggered:** P1 and P2 incidents only. The HK Tech Lead initiates within 2 business
days of resolution. Tent of Presence engineers contribute information and action items.

---

## Step 1 — Gather Raw Data First

Before writing, collect:
- Mattermost `#incidents` thread — the authoritative timeline of what was communicated and when
- Monitoring and alerting timestamps — when alerts fired, when graphs changed
- Deploy logs — what changed in the hours before the incident
- Error logs and stack traces from the impact window

Note timestamp uncertainty explicitly ("approximately 14:30 HKT based on first Mattermost
message") rather than guessing.

---

## Step 2 — Write the Post-Mortem

```
# Post-Mortem: [Short incident title]

**Incident ID:** [INC-YYYY-NNN or Sentry reference]
**Severity:** [P1 / P2]
**Date of incident:** [YYYY-MM-DD]
**Post-mortem date:** [YYYY-MM-DD]
**Author(s):** [names]
**Status:** [Draft / Final]

---

## 1. What Happened (Timeline)

| Time (HKT) | Event |
|------------|-------|
| HH:MM | First symptom observed / alert fired |
| HH:MM | Incident raised in #incidents |
| HH:MM | [Key diagnostic step] |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed / rollback initiated |
| HH:MM | Service confirmed healthy |
| HH:MM | Incident resolved |

---

## 2. Root Cause

Explain *why* the incident happened — the systemic condition that made failure possible.
Write 2–4 sentences. Avoid naming individuals. "Human error" is not a root cause — if a
human made a mistake, ask why the system allowed that mistake to cause an outage.

Weak: "A developer pushed broken code."
Strong: "The payment service had no integration tests covering the currency-conversion path.
A type mismatch passed unit tests but failed at runtime. The absence of canary deployments
meant the change reached 100% of production traffic immediately."

---

## 3. Impact

- **Who was affected:** [all users / logged-in users / specific region or feature]
- **Duration:** [HH:MM to HH:MM HKT — X minutes total]
- **Scope:** [which features, endpoints, or data were affected]
- **Volume:** [approximate users or transactions affected, if known]
- **Data integrity:** [was any data lost, duplicated, or corrupted? Yes/No — explain if yes]

---

## 4. Resolution

What stopped the incident? Be specific enough that someone who wasn't there could follow.
- What was the fix or rollback?
- Who performed it?
- Why did this fix work?
- Were there interim mitigations before the full fix?

---

## 5. Prevention (Action Items)

Each action item must become a ticket. Vague items ("improve monitoring") don't get done.
Make them specific enough to assign and estimate.

| Action | Why this prevents recurrence | Owner | Ticket |
|--------|------------------------------|-------|--------|
| [specific action] | [systemic gap it closes] | @engineer | #NNN |

Create a ticket for each row before the post-mortem is finalised.
```

---

## Step 3 — Review for Blame Language

Before sharing, read once specifically for blame. Replace language that:
- Names a person as the cause
- Uses "should have" or "failed to" in a way implying personal fault
- Assigns root cause to a human decision rather than the system that permitted it

Reframe: "the engineer forgot to update the env variable" → "the deployment checklist
did not include a step to verify environment variables before rollout."

---

## Step 4 — Create the Tickets

After the HK Tech Lead approves the draft, create a ticket for each Prevention row.
Each ticket must: reference the incident, describe the specific change, have an owner,
and be prioritised in the next sprint. Fill in ticket links in the Prevention table
before marking the post-mortem Final.

---

## Final Checklist

- [ ] Timeline uses real timestamps from logs/Mattermost, not recalled approximations
- [ ] Root cause explains the systemic condition, not the triggering action
- [ ] Impact includes duration, scope, and data integrity statement
- [ ] Every prevention action item has a corresponding ticket
- [ ] Ticket links filled into Prevention table
- [ ] Reviewed for blame language
- [ ] HK Tech Lead has approved
