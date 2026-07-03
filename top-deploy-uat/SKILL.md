---
name: top-deploy-uat
description: >
  Guide Tent of Presence engineers through deploying to the UAT (staging) environment. Use this
  whenever the user mentions deploying to staging, pushing to UAT, triggering a UAT deploy, or
  verifying changes after deployment. Also trigger when the user asks "how do I deploy?", "is my
  PR ready to go to UAT?", "how does CI/CD work?", or anything about the deployment pipeline.
  Tent of Presence engineers only have access to UAT — redirect any production deployment
  questions to top-deploy-prod.
---

# top-deploy-uat — UAT Deployment Guide

Tent of Presence engineers can deploy to UAT only. Production is owned by the HK team.
If the engineer is asking about a production deployment, redirect to `top-deploy-prod`.

---

## How UAT deployment works

Deployment is fully automated via GitHub Actions — there is no manual deploy command.
The pipeline triggers when a PR is merged into the `uat` branch.

```
Feature/fix branch → PR approved by HK engineer
  → Merge PR into uat
    → GitHub Actions deploys automatically
      → Check #deployment for result
        → Verify on staging
```

---

## Step 1 — Confirm the PR is mergeable

Before merging, confirm:

- [ ] PR has been **approved by at least one HK engineer**
- [ ] All CI checks on the PR are green
- [ ] Branch was created from `master`, not from `uat`
- [ ] No merge conflicts with `uat`

Merging a broken PR into `uat` blocks everyone else from deploying.

---

## Step 2 — Merge into `uat`

Merge via GitHub's UI or CLI using a **merge commit** (not squash, not rebase). Merge commits
preserve full history and make rollback and bisecting easier.

```bash
git checkout uat
git pull origin uat
git merge --no-ff origin/<your-branch>
git push origin uat
```

Do not force-push to `uat`. It is a shared branch — force-pushes break other engineers' local copies.

---

## Step 3 — Watch `#deployment` on Mattermost

GitHub Actions picks up the change automatically. Watch `#deployment` for the result.

**If the pipeline fails:**
- Read the failure message in `#deployment`
- Click through to the GitHub Actions run for the full log
- Fix the issue on your branch and open a new PR into `uat`
- Do not commit fixes directly to `uat` — always go through a PR

---

## Step 4 — Verify your changes on staging

After a successful deploy, open the staging environment and test your specific changes.
Do not assume the deploy worked correctly just because the pipeline passed.

- Navigate to the feature/fix area in staging
- Reproduce the scenario from your ticket
- Confirm the expected outcome
- Check for obvious regressions in nearby flows

If something looks wrong on staging, investigate before assuming it is a deployment issue
— it may be a data or configuration difference between environments.

---

## The one rule about `uat`

The `uat` branch exists only for deploying and testing. Never create feature or fix branches
from `uat`. Always branch from `master`.

Branching from `uat` inherits unreleased changes that may not go to production, causing
confusion and merge conflicts at release time.
