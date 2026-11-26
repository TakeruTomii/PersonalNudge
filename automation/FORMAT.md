# Message format (times webhook)

Compose the webhook body in **Slack mrkdwn** inside the JSON `"text"` field. If `INSTRUCTIONS.md` and this file disagree, **FORMAT.md** wins.

## Hard rules

- **Forbidden:** raw URLs like `— https://github.com/...` without `<url|label>` mrkdwn.
- **Required:** when Step 2b found `slack_permalink`, every PR line **must** include `· <permalink|Slack>` (or ja `|Slack`).

## Opening

- Start with `<@NUDGE_SLACK_USER_ID>` plus a short, light intro (`NUDGE_LOCALE` + optional `NUDGE_TONE`).
- **ja (default copy):**  
  `<@U…> お疲れ様です！代わりに現況を調べてまいりました！お手隙の時にご確認ください！（{weekday} {HH:MM} JST）`
- **en:**  
  `<@U…> Hope your day’s going well — I pulled together what’s still open for you. Peek when you have a moment! ({weekday} {HH:MM} JST)`
- **bilingual:** ja line first, then one shorter en line.

## Sections (fixed order)

Render only sections that have at least one item. Never print empty section headers.

| # | ja heading | en heading |
|---|------------|------------|
| 1 | *自動PR（Renovate・LD など）* | *Automated PRs (Renovate, LD, …)* |
| 2 | *その他レビュー（受信・依頼）* | *Other reviews (inbox & asked)* |
| 3 | *今日の当番* | *Today's duty* |
| 4 | *Slack フォロー* | *Slack follow-ups* |

`bilingual`: use ja heading, optional `(EN: …)` on the heading line only.

## Line template — PR with Slack (preferred)

GitHub link is on the **title text only** (do not add a separate `PR #n` link).

```text
• <{github_pr_url}|{line_label}> · <{slack_permalink}|Slack>
```

`line_label` = optional `[Kind]` + `[依頼]` + `{owner/repo}: {pr_title}` (≤100 chars). See INSTRUCTIONS for `[Kind]` prefixes.

## Line template — PR without Slack (only if Step 2b found nothing)

```text
• <{github_pr_url}|{line_label}>
```

## Outgoing review (you asked)

Add `[依頼]` (ja) or `[asked]` (en) inside `line_label` before the title (after `[Kind]` if any).

## Line template — non-PR items

```text
• {short_title} — <{slack_permalink}|thread>
```

- `owner/repo`: title in `line_label`; GitHub URL wraps the whole label, not PR number alone.

## Spacing

- Blank line between sections.
- No trailing blank lines at end of message.

## Payload

```json
{ "text": "<full mrkdwn body>" }
```

Keep a single `text` field (no `blocks` unless this file is extended later).