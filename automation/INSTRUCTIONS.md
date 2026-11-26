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
- **Duty / @mentions / bots:** since previous scheduled run, or last **4 hours** if unknown.
- **PR ↔ Slack enrichment (Step 2b):** search watch channels up to **14 days** back (or since PR was updated) so permalinks still attach even if the review post is older than 4h.

# Step 1 — GitHub (incoming reviews — source of truth for state)
For each repo in `NUDGE_GITHUB_REPOS`:
- List **open** PRs where `NUDGE_GH_REVIEWER` is still a requested reviewer and has **not** submitted APPROVE.
- Exclude: merged, closed, or user no longer a reviewer.
- Tag Renovate when author is `renovate[bot]` or head branch matches `renovate/*`.

GitHub decides merged/closed/approved; Slack does not override that.

# Step 1b — GitHub (outgoing — open PRs you authored)
For each repo in `NUDGE_GITHUB_REPOS`:
- List **open** PRs where **author** is `NUDGE_GH_REVIEWER` (or authored by the same human) and the PR is not merged/closed.
- Include draft PRs waiting on review.
- These are candidates for “reviews you asked for” when Step 2 or Step 2b finds a matching Slack thread or pending review activity.

# Step 2 — Slack (watched channels only)
Read only `NUDGE_WATCH_CHANNEL_IDS` (must include review/notification channels you use, e.g. `C08L2BAC525`).

**Incoming (others → you)**
- Pending review bots waiting on `NUDGE_SLACK_USER_ID`.
- Posts requesting **your** review: @you, your name, or “reviewer: you”.
- PR links in channel where GitHub still shows you as requested reviewer (confirm Step 1).

**Outgoing (you → others) — required to scan**
- Messages **posted by** `NUDGE_SLACK_USER_ID` that contain a GitHub pull URL and ask for review (e.g. レビュー, review, PTAL, 見てほしい, pending review bot you triggered).
- Threads you started on someone else’s PR announcement where you are waiting on reviewers.
- Bot messages in review channels that reference your open PR and are still unresolved.

**Duty & housekeeping** (4h window)
- Today: standup chair or alert sentinel for `NUDGE_SLACK_USER_ID`.
- LD cleanup with you as reviewer or assignee.

**Follow-ups** (4h window)
- Direct @you with a clear action (skip FYI, celebrations, resolved).

Drop items that GitHub confirms merged/closed. Do **not** drop open **outgoing** PRs just because they are absent from Step 1.

# Step 2b — Attach Slack permalinks to every PR row (mandatory)
Build candidate PR list = union of Step 1, Step 1b, and Step 2 PR links (dedupe by `owner/repo#number`).

For **each** candidate PR:
1. In **every** watch channel, search recent messages (see 14-day window) for that PR’s URL or `owner/repo#number` / `#number` with repo context.
2. Prefer: (a) your outgoing review-request post, (b) pending-review bot message, (c) latest relevant thread.
3. Store `slack_permalink` on the row when any match exists.

**Output rule:** If `slack_permalink` exists for a PR, the times message **must** include both GitHub and Slack links (see FORMAT.md). Never emit a naked `https://github.com/...` URL without also searching for Slack first.

If no Slack thread exists after search, GitHub-only line is allowed.

# Step 3 — Compose
If **no** items after Steps 1, 1b, 2 (including outgoing): exit silently.
Format strictly per **automation/FORMAT.md**. Classify:
- Section 1: Renovate (Step 1)
- Section 2: other **incoming** reviews (Step 1) and **outgoing** review requests (Step 1b + Step 2 outgoing), with outgoing title prefixed `[依頼]` / `[asked]` when `NUDGE_LOCALE` is ja / en
- Section 3: duty
- Section 4: other Slack follow-ups (no PR)

# Step 4 — Deliver (webhook ONLY)
Do NOT use Cursor "Send to Slack" or post as the Cursor app.
POST application/json to `NUDGE_SLACK_WEBHOOK_URL`, body at minimum: `{"text": "<mrkdwn body>"}`.
Use username/icon only as configured on the Slack app; do not add new env vars for display name.
The webhook must be created for the times channel; `NUDGE_TIMES_CHANNEL_ID` is for cross-check and read-scope only.