# Message format (times webhook)

Compose the webhook body in **Slack mrkdwn** inside the JSON `"text"` field. If `INSTRUCTIONS.md` and this file disagree, **FORMAT.md** wins.

## Hard rules

- **Forbidden:** raw URLs like `— https://github.com/...` without `<url|label>` mrkdwn.
- **Required:** when Step 2b found `slack_permalink`, every PR line **must** include `· <permalink|Slack>` (or ja `|Slack`).

## Opening

- Start with one line: `<@NUDGE_SLACK_USER_ID>` plus a short intro (locale from `NUDGE_LOCALE`).
- Example (ja): `<@U…> 未完了のまとめ（{weekday} {HH:MM} JST）`

## Sections (fixed order)

Render only sections that have at least one item. Never print empty section headers.

| # | ja heading | en heading |
|---|------------|------------|
| 1 | *Renovate / 優先レビュー* | *Renovate / high-priority reviews* |
| 2 | *その他レビュー（受信・依頼）* | *Other reviews (inbox & asked)* |
| 3 | *今日の当番* | *Today's duty* |
| 4 | *Slack フォロー* | *Slack follow-ups* |

`bilingual`: use ja heading, optional `(EN: …)` on the heading line only.

## Line template — PR with Slack (preferred)

```text
• {short_title} — <{github_pr_url}|PR #{n}> · <{slack_permalink}|Slack>
```

## Line template — PR without Slack (only if Step 2b found nothing)

```text
• {short_title} — <{github_pr_url}|PR #{n}>
```

## Outgoing review (you asked)

Same templates; prefix title with `[依頼]` (ja) or `[asked]` (en), e.g.:

```text
• [依頼] amptalk/analysis: my feature title — <url|PR #123> · <permalink|Slack>
```

## Line template — non-PR items

```text
• {short_title} — <{slack_permalink}|thread>
```

- `short_title`: `owner/repo`: title (≤80 chars).
- Renovate-tagged PRs go in section 1.

## Spacing

- Blank line between sections.
- No trailing blank lines at end of message.

## Payload

```json
{ "text": "<full mrkdwn body>" }
```

Keep a single `text` field (no `blocks` unless this file is extended later).