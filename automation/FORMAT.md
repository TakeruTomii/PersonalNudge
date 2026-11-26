# Message format (times webhook)

Compose the webhook body in **Slack mrkdwn** inside the JSON `"text"` field. If `INSTRUCTIONS.md` and this file disagree, **FORMAT.md** wins.

## Opening

- Start with one line: `<@NUDGE_SLACK_USER_ID>` plus a short intro (locale from `NUDGE_LOCALE`).
- Example (ja): `<@U…> 未完了のまとめ（{weekday} {HH:MM} JST）`

## Sections (fixed order)

Render only sections that have at least one item. Never print empty section headers.

| # | ja heading | en heading |
|---|------------|------------|
| 1 | *Renovate / 優先レビュー* | *Renovate / high-priority reviews* |
| 2 | *その他レビュー* | *Other review requests* |
| 3 | *今日の当番* | *Today's duty* |
| 4 | *Slack フォロー* | *Slack follow-ups* |

`bilingual`: use ja heading, optional `(EN: …)` suffix on the heading line only.

## Line template — review items (with PR)

When Step 2b attached a Slack permalink, use **dual links**:

```text
• {short_title} — <{github_pr_url}|PR #{n}> · <{slack_permalink}|Slack>
```

When there is no Slack match:

```text
• {short_title} — <{github_pr_url}|PR #{n}>
```

- `short_title`: `owner/repo`: title (≤80 chars).
- Put Renovate-tagged PRs in section 1 per INSTRUCTIONS.

## Line template — non-PR items

```text
• {short_title} — <{slack_permalink}|thread>
```

Duty / follow-ups without a PR use Slack permalink only.

## Spacing

- Blank line between sections.
- No trailing blank lines at end of message.

## Payload

```json
{ "text": "<full mrkdwn body>" }
```

Do not use `blocks` unless FORMAT is extended later; keep a single `text` field for compatibility.