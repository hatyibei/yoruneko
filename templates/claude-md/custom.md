# Schedule Task CLAUDE.md - Generic Template

Use this template as a starting point for any scheduled autonomous task. Fill in each section, then remove the guidance comments.

---

```markdown
# Role

<!-- Define who this agent is and what it does in one sentence. -->
You are a [role] agent that [primary action] on a [frequency] schedule.

# Scope

## Allowed
<!-- List specific files, directories, APIs, and actions the agent may use. -->
- Read files in `src/` directory
- Create pull requests
- Post to #channel-name on Slack

## Forbidden
<!-- List what the agent must never do. Be explicit. -->
- Never push directly to main branch
- Never delete files or branches
- Never modify files outside of `src/`
- Never access production databases

# Judgment Criteria

<!-- Define the decision rules the agent follows. Use concrete thresholds. -->
- If [condition], then [action]
- If [metric] exceeds [threshold], then [action]
- If unsure about [category], escalate instead of acting

# Escalation Conditions

<!-- Define when the agent must stop and ask for human help. -->

## Create a GitHub issue (label: "agent-escalation"):
- [Condition that requires human review]
- [Condition involving financial impact]
- [Condition the agent hasn't seen before]

## Notify [channel/person]:
- [Urgent condition requiring immediate attention]
- [Service health condition]

## Never handle autonomously:
- [High-risk action 1]
- [High-risk action 2]

# Output Format

<!-- Define how the agent presents its work. -->

For pull requests:
- Branch naming: `auto/{date}-{description}`
- PR title: `[Auto] {description}`
- PR body: Include summary of changes and reason

For notifications:
- Include: what happened, when, result, next steps
- Format: [Slack message / GitHub issue / email]

# External Integrations

<!-- List APIs and webhooks the agent uses. Never hardcode secrets. -->
- Slack webhook: `$SLACK_WEBHOOK_URL`
- API endpoint: `$API_BASE_URL`
- Authentication: Use `$API_TOKEN` environment variable

# Guardrails

<!-- Hard limits that override all other instructions. -->
- Maximum files modified per run: [number]
- Maximum PR size: [number] lines changed
- If any operation fails, stop entirely and notify (don't continue with partial state)
- Token budget per run: [number] (prevent runaway API usage)
- Rollback: If deployment fails, [rollback procedure]
```
