# Escalation Design Guide

## Why Escalation Matters

Autonomous scheduled agents will encounter situations they cannot or should not handle alone. Without escalation design, the agent either silently fails or takes incorrect action. Both are worse than doing nothing.

**Rule: Every scheduled task must define what it cannot do.**

## Escalation Levels

Design escalation in tiers, from least to most urgent:

### Level 0: Log Only
- Expected variations in normal operation
- Informational anomalies that don't require action
- Example: "API returned 3 results instead of usual 5"

### Level 1: Create Issue / Log to Channel
- Situations requiring human review but not urgently
- Failed operations that can wait until business hours
- Example: "Code review found potential security issue in PR #42"

### Level 2: Notify Team Channel
- Actionable items that need attention within hours
- Repeated failures or degraded service
- Example: "3 consecutive health check failures on staging"

### Level 3: Page On-Call / Direct Message
- Service down, data integrity risk, security incident
- Anything with financial impact or customer-facing outage
- Example: "Production API returning 500 errors for 10+ minutes"

## Defining Escalation in CLAUDE.md

Use explicit conditions, not vague guidance:

```markdown
# Escalation Conditions

## STOP and create a GitHub issue:
- Customer requests a refund (any amount)
- Error rate exceeds 5% over 15 minutes
- Any request touching billing data
- Unknown error type not in the known-errors list

## STOP and notify #ops-alerts Slack channel:
- Service health check fails 3 consecutive times
- Deployment has been in progress for > 30 minutes
- Disk usage exceeds 85%

## NEVER do (always escalate):
- Delete production data
- Modify billing records
- Send customer-facing emails without human approval
- Merge PRs that touch authentication or payments
```

## Common Escalation Triggers

### For CS Automation
- Angry/frustrated customer signals (profanity, ALL CAPS, "speak to manager")
- Billing, refund, or payment issues
- Legal or compliance questions
- Issues not matching any known category
- Customer mentions competing product or cancellation

### For Code Review
- Security vulnerabilities (SQL injection, XSS, auth bypass)
- Breaking API changes without version bump
- Changes to CI/CD pipeline or deployment config
- PRs larger than a defined threshold (e.g., 500+ lines)

### For Monitoring
- Service downtime exceeding SLA threshold
- Data anomalies suggesting corruption
- Unauthorized access patterns
- Resource exhaustion trends (projected to hit limit within hours)

### For Reporting
- Data source unavailable for more than 1 reporting cycle
- Metrics showing >2 standard deviation change
- Report generation failure

## Anti-Pattern: The Silent Agent

The worst outcome is an agent that encounters an error and does nothing:

```markdown
# BAD: No escalation defined
You are a CS agent. Read tickets and respond to them.

# GOOD: Clear boundaries
You are a CS agent. Read tickets and respond to them.
If you encounter any of the following, DO NOT respond.
Instead, create a GitHub issue with label "human-needed":
- Refund requests
- Legal questions
- Tickets with negative sentiment score
- Issues you haven't seen before
```

## Designing the Escalation Channel

Match urgency to channel:

| Urgency | Channel | Why |
|---|---|---|
| Low | GitHub Issue | Async, tracked, searchable |
| Medium | Slack/Discord channel | Visible to team, not intrusive |
| High | Slack DM / PagerDuty | Immediate attention |
| Critical | PagerDuty + Slack channel | Redundant notification |

Always include in escalation messages:
1. What happened (the trigger condition)
2. What the agent was trying to do
3. What data/context is relevant
4. What action is needed from the human
