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

## Line template (default)

One item = one bullet:

```text
• {short_title} — <{primary_url}|{label}>
```

- `short_title`: repo/PR title, duty name, or thread topic (≤80 chars).
- `primary_url`: prefer GitHub PR URL for review items; Slack permalink for duty/follow-ups without a PR.
- `label`: `PR #123` or `open` or `thread`.

## Spacing

- Blank line between sections.
- No trailing blank lines at end of message.

## Payload

```json
{ "text": "<full mrkdwn body>" }
```

Do not use `blocks` unless FORMAT is extended later; keep a single `text` field for compatibility.