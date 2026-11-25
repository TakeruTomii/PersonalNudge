# Personal Nudge

Weekdays at **10:00, 13:00, 16:00, and 18:00 JST**, a Cursor Automation checks GitHub PRs and selected Slack channels, then posts **one** message to your `#times` via your **Slack Incoming Webhook** (not Cursor’s Slack post). If nothing is pending, or today is a **Japanese public holiday**, it stays silent.

**Config lives only on GitHub** — this repo’s **Settings → Secrets and variables → Actions**. Do not duplicate `NUDGE_*` in the Cursor Environment UI. Each run loads values with `gh variable get` (see [automation/INSTRUCTIONS.md](automation/INSTRUCTIONS.md)).

> **Webhook URL:** GitHub **Secrets** cannot be read back by the agent. Put `NUDGE_SLACK_WEBHOOK_URL` in **Actions → Variables** on this **private** repo (not Secrets), or the run cannot POST.

## 1. Slack webhook

Slack app → **Incoming Webhooks** → webhook for **your `#times`**. Save the URL as GitHub variable `NUDGE_SLACK_WEBHOOK_URL`. Note your `U…` user ID and `C…` channel IDs.

## 2. GitHub Actions variables (this repo)

**Settings → Secrets and variables → Actions → Variables** — add:

| Variable | Value |
|----------|--------|
| `NUDGE_LOCALE` | `ja` · `en` · `bilingual` |
| `NUDGE_GH_REVIEWER` | GitHub username |
| `NUDGE_SLACK_USER_ID` | `U…` |
| `NUDGE_TIMES_CHANNEL_ID` | `#times` `C…` |
| `NUDGE_WATCH_CHANNEL_IDS` | comma-separated `C…` |
| `NUDGE_GITHUB_REPOS` | `owner/repo,...` |
| `NUDGE_TONE` | optional |
| `NUDGE_SLACK_WEBHOOK_URL` | Incoming Webhook URL |

## 3. Cursor Automation

1. **Repository** — this repo (`PersonalNudge`, default branch).
2. **Schedule (UTC cron)** — `0 1,4,7,9 * * 1-5` (Mon–Fri 10 / 13 / 16 / 18 JST).
3. **Environment** — default for this repo is fine; you do **not** need to enter `NUDGE_*` there.
4. **Tools** — GitHub + Slack **read**; **Send to Slack off**.
5. **Instructions** — paste only:

   ```
   Read and follow automation/INSTRUCTIONS.md in this repository. Execute it exactly. Do not use Send to Slack.
   ```

## 4. Smoke test

Test run → your Slack app posts. Reviewer PR → listed. All clear → silent.

## 5. Sharing

Fork or duplicate the automation; each person sets their own GitHub Variables on their fork.

**Troubleshooting:** Empty config → `gh` not authorized or variable names typo. No post → webhook in Secrets instead of Variables. Cursor-branded post → Send to Slack still on.