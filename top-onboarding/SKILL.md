---
name: top-onboarding
description: >
  Run the Tent of Presence new engineer onboarding checklist. Use this whenever an HK Tech Lead
  (or anyone with onboarding responsibility) mentions adding a new engineer, setting up a new
  joiner, preparing for someone's first day, or asking what needs to be done before a new team
  member starts. Trigger when the user says "we have a new hire", "someone is joining the team",
  "what do I need to set up for the new dev?", or "new engineer onboarding checklist". This skill
  tracks every pre-day-1 and day-1 action so nothing is missed.
---

# top-onboarding — New Engineer Onboarding Runbook

This skill is for the **HK Tech Lead** running the onboarding checklist for a new Tent of
Presence engineer. Ask for the new engineer's name, email, and start date before proceeding.

---

## Before Day 1 — HK Tech Lead actions

### Mattermost
- [ ] Create a Mattermost account for the new engineer
- [ ] Add to channels: `#town-square`, `#check-in`, `#engineering`, `#incidents`, `#deployment`
- [ ] Share the Monday 10am Google Meet link in a DM

Getting them into the right channels before Day 1 means they can read recent context rather
than starting cold.

### GitHub
- [ ] Send GitHub org invite — as **member**, not owner
- [ ] Grant minimum required permissions on relevant repositories
- [ ] Add to the GitHub Projects board
- [ ] Confirm branch protection is active on `master` and `uat`
- [ ] **Enforce 2FA before granting access** — account must have 2FA enabled first

Minimum permissions reduce blast radius if credentials are compromised. 2FA is non-negotiable.

### Clockify
- [ ] Create a Clockify account and assign to the correct workspace and project(s)

### First ticket
- [ ] Create a well-scoped onboarding ticket targeted at **Day 3**

Day 3 gives the engineer time to complete onboarding, read the working guide, and get their
local environment running before picking up real work.

### AWS (staging only)
- [ ] Create an **individual** IAM user — never a shared account
- [ ] Grant read-only CloudWatch access on **staging environment only**
- [ ] Enable MFA on the IAM user before sharing credentials
- [ ] **No production AWS access — ever.** No wildcard (`*`) permissions.

Individual IAM users allow clean audit and revocation when someone leaves. Staging-only
access means a compromised credential cannot touch production.

### Bitwarden
- [ ] Add to team vault with access to **relevant collections only**

Secrets are never shared via Mattermost or email. Bitwarden is the only approved channel.
If a secret cannot be shared via Bitwarden, escalate — do not use a workaround.

### Local environment
- [ ] Follow the README local setup on a clean machine to confirm it works end-to-end

A broken README on Day 1 kills momentum. Fix it before they arrive.

### Orientation
- [ ] Schedule a **60-minute orientation session** for Day 1 (Google Meet)

---

## Day 1 — HK Tech Lead sends

Send all of these in one Mattermost DM at the start of their first day:

- [ ] Mattermost credentials (temporary password)
- [ ] Bitwarden invite link
- [ ] GitHub org invite link
- [ ] Clockify invite link

---

## Day 1 — New engineer actions

Share this section with the new engineer:

- [ ] Sign in to Mattermost with provided credentials
- [ ] Update Mattermost profile: full name, role, timezone → **Asia/Kuala_Lumpur**
- [ ] Enable 2FA on GitHub (required before repo access is granted)
- [ ] Read the working guide in full

Reading the working guide on Day 1 is not optional — it covers branching conventions, PR
process, deployment rules, and communication norms needed from their very first PR.

---

## Onboarding status summary template

Use this when reporting status or following up:

```
Onboarding status for [Name] — start date: [DATE]

Pre-Day 1:
  Mattermost account + channels      [done / pending]
  GitHub invite + 2FA enforced       [done / pending]
  Clockify account                   [done / pending]
  First ticket (Day 3)               [done / pending]
  AWS IAM user (staging only, MFA)   [done / pending]
  Bitwarden vault access             [done / pending]
  README verified on clean machine   [done / pending]
  Orientation session scheduled      [done / pending]

Day 1 (HK Tech Lead):
  Credentials + invites sent         [done / pending]

Day 1 (new engineer):
  Mattermost profile set             [done / pending]
  GitHub 2FA enabled                 [done / pending]
  Working guide read                 [done / pending]

Blockers: [list any items that will delay access]
```
