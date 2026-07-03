# top-skills

Claude Code skills for Tent of Presence engineers — built for consistency across the team, grounded in the working guide.

---

## Purpose

Every developer on the team follows the same branching strategy, commit format, PR checklist, and incident response process. These skills encode those rules directly into Claude Code so the right guidance is always one natural-language prompt away — no need to re-read the working guide before every PR or branch.

**Primary reference:** All skills are derived from `docs/working-guide.md` in the `engineering` repository. When the working guide is updated, the relevant skills in this repo should be updated to match.

---

## Skills

| Skill | What it does |
|---|---|
| `top-check-in` | Generates your daily standup from real GitHub activity — commits, PRs, and issues |
| `top-pr` | Walks you through creating a PR that passes the team's full working-guide checklist |
| `top-branch` | Validates branch names, explains branching rules, and guides correct branch setup |
| `top-commit` | Guides you through writing a Conventional Commit with the right type, scope, and format |
| `top-security` | Runs an OWASP-based security checklist on your code (Node.js, Express, MySQL, React) |
| `top-incident` | Assigns severity, tells you exactly what to post in `#incidents`, and who to notify |
| `top-deploy-uat` | Guides Tent of Presence engineers through a UAT deployment end-to-end |

---

## Installation

Skills work at **project level** — they live inside `.claude/skills/` in a repository and are only active when Claude Code is opened from that project directory.

### One-time setup per project

Run this from your project root:

```bash
# Create the skills directory if it doesn't exist
mkdir -p .claude/skills

# Clone top-skills into a temp location
git clone git@github.com:zach-tenthub/top-skills.git /tmp/top-skills

# Copy all 7 skills into the project
cp -r /tmp/top-skills/top-* .claude/skills/

# Commit so every team member gets them automatically on clone
git add .claude/skills/
git commit -m "chore: add top-skills Claude Code skills"
git push
```

Committing `.claude/skills/` into the repo means every developer who clones the project gets all skills immediately — no individual setup step needed.

### Verify the skills are loaded

Open Claude Code in the project directory and run:

```
/skills
```

All 7 `top-*` skills should appear in the list. If a skill is missing, check that its folder exists at `.claude/skills/<skill-name>/SKILL.md`.

---

## Usage

Skills are triggered by natural language — just describe what you need:

| What you say | Skill triggered |
|---|---|
| `generate my daily check-in` | `top-check-in` |
| `I'm ready to open a PR` | `top-pr` |
| `I need to start a new feature, what branch should I create?` | `top-branch` |
| `help me write a commit message for my fix` | `top-commit` |
| `do a security check on this endpoint` | `top-security` |
| `something is broken in production, what do I do?` | `top-incident` |
| `I want to deploy my changes to UAT` | `top-deploy-uat` |

You don't need to type the skill name — Claude Code reads each skill's description and picks the best match based on what you wrote.

---

## Keeping Skills Up to Date

When the working guide changes, update the affected skills and push here. Then pull into each project:

```bash
# Pull latest from top-skills
git clone git@github.com:zach-tenthub/top-skills.git /tmp/top-skills

# Overwrite skills in your project
cp -r /tmp/top-skills/top-* .claude/skills/

# Commit the update
git add .claude/skills/
git commit -m "chore: update top-skills to latest"
git push
```

---

## Contributing

If the working guide is updated and a skill needs to change, edit the relevant `SKILL.md` in this repo and open a PR against `main`. The `description` field at the top of each `SKILL.md` controls when Claude triggers the skill — keep it specific and include real-world phrases a developer would actually say.
