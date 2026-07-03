---
name: top-check-in
description: |
  Auto-generate daily work check-ins by querying your GitHub activity (commits, PRs, issues).
  Triggers when you mention a developer's name or username. Pulls real work from GitHub commits 
  and open tasks to populate yesterday/today sections — zero manual typing needed.
  Outputs professional, ready-to-share check-in for Mattermost, email, or team reports.
  Use this whenever a developer needs a daily status update, or to automate team check-ins.
  Requires the GitHub MCP to be connected. Groups work by project/repository.
  Auto-detects active sprint milestones — no extra typing needed from the developer.
---

# GitHub-Powered Daily Check-in Generator

## Overview

Generates professional daily work check-ins from real GitHub activity using the connected GitHub MCP.

**Schedule is fully automatic:**
- **Start time** = moment the developer triggers this skill (Asia/Shanghai timezone)
- **End time** = start time + 9 hours

**Check-in content — grouped by repository:**
- **Point 3 — Yesterday:** Claude-summarized bullets per repo, from commits + issues closed yesterday
- **Point 4 — Today:** Open PRs (with real review status) and open issues per repo, split into **Sprint** (has milestone) and **Backlog** (no milestone). Issues covered by an open PR are suppressed. Issues handed off to another developer are flagged.

---

## Implementation: Step-by-Step

### Step 1 — Extract Username

Parse the name/username from the user's message.

### Step 2 — Capture Trigger Time

Run this immediately before anything else:

```bash
python3 -c "
from datetime import datetime, timedelta
import pytz
tz = pytz.timezone('Asia/Shanghai')
now = datetime.now(tz)
end = now + timedelta(hours=9)

def fmt(dt):
    h = dt.strftime('%I').lstrip('0') or '12'
    return h + ':' + dt.strftime('%M') + dt.strftime('%p').lower()

print('today:', now.strftime('%Y.%m.%d'))
print('yesterday:', (now - timedelta(days=1)).strftime('%Y-%m-%d'))
print('start_time:', fmt(now))
print('end_time:', fmt(end))
"
```

### Step 3 — Resolve GitHub Login

Call `get_me` to get the authenticated `login`. Use it for all queries below.

### Step 4 — Fetch GitHub Data

**A. Yesterday's commits** — `search_commits`:
```
query: "author:{login} committer-date:>={YESTERDAY_DATE}"
sort: committer-date, order: desc, perPage: 30
```
Collect: `commit.message` (full text) and `repository.full_name`.

**B. Issues closed yesterday** — `search_issues`:
```
query: "assignee:{login} is:issue is:closed closed:>={YESTERDAY_DATE}"
sort: updated, order: desc, perPage: 20
```
Collect: `.title`, `#number`, `repository_url`.

**C. Open PRs** — run both queries and merge, deduplicating by PR number:
```
query 1: "assignee:{login} is:open is:pr"
query 2: "author:{login} is:open is:pr"
sort: updated, order: desc, perPage: 20 each
```
Developers often open PRs without self-assigning — querying both author and assignee ensures nothing is missed. Collect: `.title`, `#number`, `repository_url`, `.draft`, `.body`.

**D. Open issues** — `search_issues`:
```
query: "assignee:{login} is:open is:issue"
sort: updated, order: desc, perPage: 20
```
Collect: `.title`, `#number`, `repository_url`, `.milestone.title` (null if not set).

### Step 5 — Enrich PRs with Review Status

For each unique PR from Step 4C, call `pull_request_read` (method: `get_reviews`). Derive status:

- `draft: true` → **"🚧 draft — not ready for review"** (skip review lookup)
- Any review state `CHANGES_REQUESTED` → **"🔁 changes requested"**
- At least one `APPROVED`, none `CHANGES_REQUESTED` → **"✅ approved"**
- No human reviews (empty array, or bot-only) → **"🔄 awaiting review"**

Store the derived status on each PR object.

### Step 6 — Deduplicate Issues Covered by PRs

Scan each PR's `.body` for closing keywords (case-insensitive):
`closes #N`, `fixes #N`, `resolves #N`, `close #N`, `fix #N`, `resolve #N`

Build a suppressed set per repo. Issues in this set are omitted from Point 4.

### Step 7 — Detect Handed-Off Issues

For each remaining open issue (after Step 6), call `issue_read` (method: `get_comments`) and read the most recent comment. Flag the issue as handed off if the latest comment contains any of:

- Statements indicating backend/investigation is complete and another role needs to act: "backend is ready", "backend done", "no backend change needed", "frontend only", "this is a frontend issue", "investigation complete"
- The developer explicitly directing another person: mentions `@someone` alongside "you can start", "please handle", "take over", "your turn", "handed off to"
- The developer themselves explaining it is no longer their responsibility

Handed-off issues are **not suppressed** — they stay in Point 4 but get a `⚠️ handed off — consider unassigning` note so the developer is reminded to clean up GitHub.

### Step 8 — Group by Repository

Group all results by `repo` (owner/name). For open issues, split by milestone:

- `.milestone` not null → `sprint_issues`
- `.milestone` null → `backlog_issues`
- Suppressed issues → excluded
- Handed-off issues → included with flag

Sort repos by activity volume (most commits first).

### Step 9 — Summarize Yesterday's Work Per Repo

For each repo in `yesterday_by_repo`, summarize commits and closed issues into 2-4 concise bullets:

- Group related commits into a single bullet
- Use plain standup language — what was done, not how
- Prefix closed issues as: `- Closed issue #N: {summary}`
- Aim for 2-4 bullets per repo; never more than 5

### Step 10 — Format Today's Pending Work Per Repo

For each repo in `today_by_repo`:
- List PRs first with real review status emoji
- If a repo has BOTH sprint and backlog issues, add `Sprint:` and `Backlog:` markers
- If only one group, skip the markers
- Handed-off issues: append `⚠️ handed off — consider unassigning`

### Step 11 — Build the Check-in

Use this exact template:

```
**{DisplayName}'s Daily Check-in — {YYYY.MM.DD}**
Flexible schedule: {start_time} - {end_time}

**1. Are you on schedule?**
Yes

**2. Any roadblocks?**
No

**3. What did you do yesterday?**

*{repo-1}*
- {summarized bullet}
- {summarized bullet}

*{repo-2}*
- {summarized bullet}

**4. What will you do today?**

*{repo-1}*
- PR #N: {title} 🔄 awaiting review
Sprint:
- Issue #N: {title}
Backlog:
- Issue #N: {title} ⚠️ handed off — consider unassigning

*{repo-2}*
- Issue #N: {title}
```

Fallbacks:
- No activity in a repo for yesterday → skip that repo from point 3
- No open tasks in a repo for today → skip that repo from point 4
- No yesterday activity at all → `- No activity found for yesterday`
- No today tasks at all → `- No open tasks found`
- All issues have milestones → no Backlog section
- No issues have milestones → no Sprint section (list all as-is)

### Step 12 — Output

Print the check-in in a code block for easy copying. Follow with:
"Ready to paste into Mattermost, email, or your team report."

---

## Error Handling

| Situation | Response |
|-----------|----------|
| GitHub MCP not connected | "Please connect the GitHub MCP first via Settings → Connections." |
| No commits or closed issues | `- No activity found for yesterday` in point 3 |
| No open PRs or issues | `- No open tasks found` in point 4 |
| `pull_request_read` fails for a PR | Fall back to `🔄 (under review)` for that PR |
| `issue_read` comments fail | Skip handoff detection for that issue |
| Rate limit | "GitHub API rate limit hit — try again in ~1 hour." |
| Wrong username | Ask user to confirm their exact GitHub login. |

---

## Tips

- **Run at the start of your day** — trigger time = clock-in, +9h = clock-out
- **Write descriptive commit messages** — they feed the summarizer; more context = better bullets
- **Close issues when done** — closed issues appear under the right repo in point 3
- **Assign yourself to PRs/issues** — both authored and assigned PRs are captured, but self-assigning keeps GitHub state clean
- **Use closing keywords in PR descriptions** — `Closes #N` suppresses the linked issue automatically
- **Unassign yourself from handed-off issues** — the skill flags them with ⚠️ as a reminder
- **Set milestones on sprint issues** — the skill auto-detects them; developers type nothing extra
- **Multiple projects** — each repo gets its own section automatically; no setup needed
