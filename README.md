# Personal Nudge

Weekdays at **10:00, 13:00, 16:00, and 18:00 JST**, a Cursor Automation checks GitHub PRs and selected Slack channels, then posts **one** message to your `#times` via your **Slack Incoming Webhook** (not Cursor’s Slack post). If nothing is pending, or today is a **Japanese public holiday**, it stays silent.

`NUDGE_*` on **GitHub Actions** Variables/Secrets and the same names on your **Cursor Environment**. Behavior is defined in **[automation/INSTRUCTIONS.md](automation/INSTRUCTIONS.md)** only.

## 1. Slack webhook

Slack app → **Incoming Webhooks** → webhook for **your `#times`**. Secret: `NUDGE_SLACK_WEBHOOK_URL`. Note your `U…` user ID and `C…` channel IDs (times + channels to watch).

## 2. GitHub → Settings → Secrets and variables → Actions

| Name | Type | Value |
|------|------|--------|
| `NUDGE_LOCALE` | Variable | `ja` · `en` · `bilingual` |
| `NUDGE_GH_REVIEWER` | Variable | GitHub username |
| `NUDGE_SLACK_USER_ID` | Variable | `U…` |
| `NUDGE_TIMES_CHANNEL_ID` | Variable | `#times` `C…` |
| `NUDGE_WATCH_CHANNEL_IDS` | Variable | comma-separated `C…` |
| `NUDGE_GITHUB_REPOS` | Variable | `owner/repo,...` |
| `NUDGE_TONE` | Variable | optional |
| `NUDGE_SLACK_WEBHOOK_URL` | Secret | webhook URL |

Copy the same keys into Cursor **Environment** (e.g. `personal-nudge`); disable **Skip Install** if secrets do not appear.

## 3. Cursor Automation

1. **Repository** — connect this repo (`PersonalNudge`, default branch). The agent checks out the repo to read the prompt file.
2. **Schedule (UTC cron)** — `0 1,4,7,9 * * 1-5` (Mon–Fri 10 / 13 / 16 / 18 JST).
3. **Environment** — your env with `NUDGE_*`.
4. **Tools** — enable GitHub and Slack **read** as needed; keep **Send to Slack off** (webhook in the prompt posts to `#times`).
5. **Instructions** — paste **only** this into the automation’s Instructions field (not the contents of `INSTRUCTIONS.md`):

   ```
   Read and follow automation/INSTRUCTIONS.md in this repository. Execute it exactly. Do not use Send to Slack.
   ```

   If the editor supports `@` file picks with the repo attached, you can use `@automation/INSTRUCTIONS.md` with the same meaning. Edit behavior by pushing changes to that file.

## 4. Smoke test

Test run → message from **your Slack app**. Open PR with you as reviewer → listed. Merge/approve with nothing else open → silent run.

## 5. Sharing

Duplicate the automation; each person uses their own webhook and `NUDGE_*`. Same Instructions line; fork or share this repo if the prompt differs per team.

**Troubleshooting:** Posts from Cursor → Send to Slack still on. Stale PRs → GitHub access broken. Wrong Slack items → fix `NUDGE_WATCH_CHANNEL_IDS` and Slack read scope in the editor.