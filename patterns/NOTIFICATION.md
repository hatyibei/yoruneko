# Notification Design Guide

## Principles

1. **Every scheduled task needs notification** - Silent automation is invisible automation
2. **Success and failure need different channels** - Don't spam success into alert channels
3. **Frequency control** - Match notification frequency to action frequency
4. **Content over noise** - Include actionable information, not raw logs

## Notification Matrix

| Event | Channel | Frequency | Content |
|---|---|---|---|
| Task completed successfully | Low-priority channel / log | Every run or daily digest | Summary of actions taken |
| Task completed with warnings | Team channel | Every occurrence | Warning details + context |
| Task failed | Alert channel | Every occurrence | Error + what to do |
| Task needs escalation | DM / PagerDuty | Immediately | Full context + required action |

## Slack Webhook Integration

### Setup
1. Create a Slack App or use Incoming Webhooks
2. Store webhook URL as a secret (never in CLAUDE.md)
3. Reference via environment variable

### CLAUDE.md Pattern
```markdown
# Notifications

After each run, send a summary to Slack:

- Success: POST to $SLACK_WEBHOOK_SUCCESS
  Format: "CS Agent: Processed {n} tickets. {resolved} resolved, {escalated} escalated."

- Failure: POST to $SLACK_WEBHOOK_ALERT
  Format: "CS Agent ERROR: {error_message}. Last successful run: {timestamp}."

Use this curl pattern:
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"your message here"}' \
  "$SLACK_WEBHOOK_URL"
```

## Discord Webhook Integration

Similar to Slack but with Discord's embed format:

```markdown
# Notifications

Send results to Discord webhook:
curl -X POST -H 'Content-type: application/json' \
  --data '{"content":"your message here"}' \
  "$DISCORD_WEBHOOK_URL"

For rich embeds:
curl -X POST -H 'Content-type: application/json' \
  --data '{"embeds":[{"title":"Daily Report","description":"...","color":5763719}]}' \
  "$DISCORD_WEBHOOK_URL"
```

## Email Notification (via API)

For critical notifications that shouldn't be missed:

```markdown
# Email Notifications

For critical escalations, send email via SendGrid API:
curl -X POST https://api.sendgrid.com/v3/mail/send \
  -H "Authorization: Bearer $SENDGRID_API_KEY" \
  -H "Content-Type: application/json" \
  --data '{
    "personalizations":[{"to":[{"email":"oncall@company.com"}]}],
    "from":{"email":"agent@company.com"},
    "subject":"[ESCALATION] ...",
    "content":[{"type":"text/plain","value":"..."}]
  }'
```

## Notification Frequency Control

### Digest Pattern
Instead of notifying on every run, accumulate results and send a digest:

```markdown
# Notification Rules
- Errors: Notify immediately on every occurrence
- Warnings: Notify immediately, but deduplicate (same warning within 1 hour = skip)
- Success: Send daily digest at 09:00 JST summarizing all runs in the past 24 hours
```

### Avoiding Notification Fatigue
- Use threads (Slack) for related notifications
- Aggregate "nothing happened" into daily digest
- Never notify about expected no-ops ("No new tickets to process" = skip notification)
- Reserve DMs/pages for genuinely urgent items

## GitHub Actions Specific

For GitHub Actions schedule triggers, use these built-in notification methods:

1. **GitHub Issues**: Create an issue for escalation (tracked, searchable)
2. **PR Comments**: Comment on relevant PRs for review-related notifications
3. **Job Summary**: Use `$GITHUB_STEP_SUMMARY` for run-level reporting
4. **Repository Dispatch**: Trigger downstream workflows on specific events

## Notification Content Template

Every notification should answer:

```
WHAT: [One-line summary of what happened]
WHEN: [Timestamp of the event]
RESULT: [Success / Warning / Failure / Escalation]
DETAILS: [2-3 lines of specifics]
ACTION: [What the recipient should do, if anything]
LINK: [URL to relevant PR, issue, dashboard, or logs]
```
