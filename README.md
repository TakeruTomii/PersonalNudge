# Personal Nudge

Weekdays at **10:00, 13:00, 16:00, and 18:00 JST**, a Cursor Automation checks GitHub PRs and selected Slack channels, then posts **one** message to your `#times` via your **Slack Incoming Webhook** (not Cursor’s Slack post). If nothing is pending, or today is a **Japanese public holiday**, it stays silent.

**Values:** edit only on GitHub — this repo’s **Settings → Secrets and variables → Actions → Variables** (the **Variables** tab, not Environments, not Secrets).  
**Runtime:** Cursor cannot read those with its built-in `gh` (you may see `403 Resource not accessible by integration`). Add one **Runtime Secret** in Cursor so the agent can call the GitHub API (below).

## 1. Slack webhook

Slack app → **Incoming Webhooks** → webhook for **your `#times`**. Save the URL as GitHub variable `NUDGE_SLACK_WEBHOOK_URL`. Note your `U…` user ID and `C…` channel IDs.

## 2. GitHub Actions variables (this repo)

| Variable | Value |
|----------|--------|
| `NUDGE_LOCALE` | `ja` · `en` · `bilingual` |
| `NUDGE_GH_REVIEWER` | GitHub username |
| `NUDGE_SLACK_USER_ID` | `U…` |
| `NUDGE_TIMES_CHANNEL_ID` | `#times` `C…` |
| `NUDGE_WATCH_CHANNEL_IDS` | comma-separated `C…` |
| `NUDGE_GITHUB_REPOS` | e.g. `amptalk/analysis` |
| `NUDGE_TONE` | optional |
| `NUDGE_SLACK_WEBHOOK_URL` | Incoming Webhook URL |

**Not** under GitHub **Environments** (`production`, etc.) — the agent only reads **repository** Variables for `TakeruTomii/PersonalNudge`.

Check locally (your user `gh`, not Cursor’s):

```bash
gh variable get NUDGE_LOCALE --repo TakeruTomii/PersonalNudge
```

If that works on your laptop but the automation fails, the fix is step 3 (PAT), not re-typing variables.

## 3. Cursor — one secret for GitHub read

1. **Fine-grained PAT** ([GitHub → Settings → Developer settings → Fine-grained tokens](https://github.com/settings/personal-access-tokens/new))  
   - Repository access: **Only** `PersonalNudge`  
   - Permissions: **Metadata: Read-only** and **Variables: Read-only** (`actions_variables=read`)  
   - **Not** enough: **Actions** read alone — that does **not** grant Actions Variables API access (403).

2. **Cursor** → create or open a Cloud **Environment** → **Runtime Secret**  
   - Name: `NUDGE_GH_VARIABLES_TOKEN` (exact spelling)  
   - Value: the PAT **string shown once** when you generate the token (starts with `github_pat_…`)

   If you **edit** a fine-grained PAT’s permissions, GitHub may issue a **new** string — you must **paste that new string** here again (old string → **401 Bad credentials**). Already-running setup VMs keep the old value until you save and start a new run.

   Do **not** duplicate every `NUDGE_*` in Cursor — only this token.

3. In the **Automation** editor, under **Environment**, select that Environment (not “no environment”). **Skip Install** must be **off** or secrets never inject.

4. PAT value does **not** go in the Instructions text — only the secret name above. The agent reads it from the VM environment.

## 3b. Cloud Environment setup (optional)

This repo has **no app build** (README + prompt only). Guided setup may pause on “install”; you can **save the Environment** once checkout works and `NUDGE_GH_VARIABLES_TOKEN` is set. A snapshot is optional.

Verify PAT before Save:

```bash
curl -s -H "Authorization: Bearer $NUDGE_GH_VARIABLES_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/TakeruTomii/PersonalNudge/actions/variables/NUDGE_LOCALE
```

Expect JSON with `"value"` — not 403.

## 4. Cursor Automation

1. **Repository** — `PersonalNudge`, default branch.  
2. **Environment** — the one that has Runtime Secret `NUDGE_GH_VARIABLES_TOKEN`.  
3. **Schedule (UTC)** — `0 1,4,7,9 * * 1-5` (Mon–Fri 10 / 13 / 16 / 18 JST).  
4. **Tools** — GitHub + Slack read; **Send to Slack off**.  
5. **Instructions** — paste (PAT value **never** here):

   ```
   Read and follow automation/INSTRUCTIONS.md in this repository. Execute it exactly. Do not use Send to Slack. Load NUDGE_* via GitHub REST using the NUDGE_GH_VARIABLES_TOKEN environment variable; do not use gh variable get.
   ```

   After changing this block or `INSTRUCTIONS.md`, run once manually so the checkout picks up `main`.

## 5. Smoke test

Test run → Slack post from **your app**. On 403 in logs → PAT missing or wrong scope. Auth OK + open reviewer PR → listed in nudge.

## 6. Sharing

Fork; each person sets their own GitHub Variables + their own `NUDGE_GH_VARIABLES_TOKEN` on their fork.

**Troubleshooting**

| Symptom | Fix |
|---------|-----|
| 403 on variable read | PAT needs **Variables: Read-only**, not only Actions read; update secret if token string changed |
| 403 with Metadata OK | Same as above — add **Variables: Read-only** on the fine-grained PAT |
| **401** Bad credentials | PAT rotated or revoked — generate/copy token again → update Runtime Secret → new run |
| Log says token unset | Environment not selected on automation, or Skip Install on, or secret name typo |
| Still uses `gh variable get` | Update editor Instructions line + latest `main` `INSTRUCTIONS.md` |
| Silent run, vars set | Was auth failure treated as empty — check run log after prompt update |
| Vars only under **Environments** | Move to **Actions → Variables** (repo tab) |
| Webhook only in **Secrets** | Move URL to **Variables** (Secrets are not readable) |
| Cursor-branded Slack post | Send to Slack still on |