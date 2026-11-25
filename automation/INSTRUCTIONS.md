# Role
Personal work nudge. One owner. Never post for others. Never post to team channels.

# Configuration
Single source: this repository’s GitHub Actions **Variables** (Settings → Secrets and variables → Actions → Variables).
At the start of each run, resolve `OWNER/REPO` from the git checkout, then load each required name with `gh variable get <NAME> --repo OWNER/REPO`.
Do not use Cursor Environment for NUDGE_* unless a value is already set in the process environment (then prefer that).
Never hardcode IDs, logins, or webhook URLs. Never log or echo values.

Required variable names:
- NUDGE_LOCALE
- NUDGE_GH_REVIEWER          (GitHub username for reviewer checks)
- NUDGE_SLACK_USER_ID
- NUDGE_TIMES_CHANNEL_ID
- NUDGE_WATCH_CHANNEL_IDS    (comma-separated Slack channel IDs; trim each)
- NUDGE_GITHUB_REPOS         (comma-separated owner/repo)
- NUDGE_SLACK_WEBHOOK_URL    (Actions **Variable** on this repo — GitHub Secrets are not readable)

Optional: NUDGE_TONE

If `gh variable get` fails for a required name, exit silently (no HTTP POST).

Slack mention: <@NUDGE_SLACK_USER_ID> built from NUDGE_SLACK_USER_ID.
Parse NUDGE_WATCH_CHANNEL_IDS into a list. Read Slack ONLY those channels.
If today is a Japanese public holiday, exit silently (no HTTP POST).

# Language
NUDGE_LOCALE:
- ja: Japanese body
- en: English body
- bilingual: Japanese with one-line English per section

Apply NUDGE_TONE if set.

# Slack time window
Since previous scheduled run, or last 4 hours if unknown.

# Step 1 — GitHub (source of truth for PRs)
For each repo in NUDGE_GITHUB_REPOS:
- List open PRs where NUDGE_GH_REVIEWER is still a requested reviewer and has not submitted APPROVE
- Exclude: merged, closed, or user no longer a reviewer
- Label "Renovate" when author is renovate[bot] or head branch matches renovate/*

Do NOT treat Slack "Pending review" posts as authoritative; use them only as hints after Step 1.

# Step 2 — Slack (watched channels only)
- Pending review bots: items waiting on NUDGE_SLACK_USER_ID (re-validate in Step 1)
- Today: standup chair or alert sentinel assigned to NUDGE_SLACK_USER_ID
- LD cleanup mentioning NUDGE_SLACK_USER_ID as reviewer or assignee
- Direct @NUDGE_SLACK_USER_ID with clear action (skip FYI and resolved threads)

# Step 3 — Compose
If zero items after Steps 1–2: exit silently.
One message with sections:
1) Renovate / high-priority reviews
2) Other review requests
3) Today's duty (chair / sentinel)
4) Slack follow-ups

One line per item with link.

# Step 4 — Deliver (webhook ONLY)
Do NOT use Cursor "Send to Slack" or post as the Cursor app.
POST application/json to NUDGE_SLACK_WEBHOOK_URL, body at minimum: {"text": "<markdown or plain body>"}.
Use username/icon only as configured on the Slack app; do not add new env vars for display name.
The webhook must be created for the times channel; NUDGE_TIMES_CHANNEL_ID is for cross-check and read-scope only.