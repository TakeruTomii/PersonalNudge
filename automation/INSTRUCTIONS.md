# Role
Personal work nudge. One owner. Never post for others. Never post to team channels.

# Configuration
Single source for values: this repo’s **Actions → Variables** tab (Settings → Secrets and variables → Actions → **Variables** — not Environments, not Secrets).

## Bootstrap (before Step 1 — do not skip)
In Cursor Automations, **do not** load config with `gh variable get` or unauthenticated `gh api`; the integration token returns 403.

1. Confirm `NUDGE_GH_VARIABLES_TOKEN` is set in the process environment (Cursor Runtime Secret on the automation’s **attached Environment**). If unset, stop without Webhook POST and report: `NUDGE_GH_VARIABLES_TOKEN missing — attach Environment with Runtime Secret; disable Skip Install on the automation`.
2. Resolve `OWNER/REPO` from the git checkout (e.g. `TakeruTomii/PersonalNudge`).
3. For each required `NUDGE_*` name below: if already set in the environment, use it; otherwise  
   `GET https://api.github.com/repos/OWNER/REPO/actions/variables/{NAME}`  
   with headers `Authorization: Bearer $NUDGE_GH_VARIABLES_TOKEN` and `Accept: application/vnd.github+json`, parse JSON field `value`.

Never hardcode IDs, logins, or webhook URLs. Never log or echo variable values or the PAT.

Required names:
- NUDGE_LOCALE
- NUDGE_GH_REVIEWER          (GitHub username for reviewer checks)
- NUDGE_SLACK_USER_ID
- NUDGE_TIMES_CHANNEL_ID
- NUDGE_WATCH_CHANNEL_IDS    (comma-separated Slack channel IDs; trim each)
- NUDGE_GITHUB_REPOS         (comma-separated owner/repo)
- NUDGE_SLACK_WEBHOOK_URL    (Actions Variable on this repo)

Optional: NUDGE_TONE

If any variable GET returns **401 or 403**, stop without Webhook POST; report that the fine-grained PAT needs **Metadata: Read-only** and **Variables: Read-only** on this repo (**Actions** read alone is insufficient).
If **404** or empty value for a required name after the REST calls, exit silently (no HTTP POST).

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
Format the message strictly per **automation/FORMAT.md** (read that file from this repo).

# Step 4 — Deliver (webhook ONLY)
Do NOT use Cursor "Send to Slack" or post as the Cursor app.
POST application/json to NUDGE_SLACK_WEBHOOK_URL, body at minimum: {"text": "<markdown or plain body>"}.
Use username/icon only as configured on the Slack app; do not add new env vars for display name.
The webhook must be created for the times channel; NUDGE_TIMES_CHANNEL_ID is for cross-check and read-scope only.