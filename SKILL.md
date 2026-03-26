---
name: yoruneko
description: Design and set up Claude Code scheduled tasks. Use when the user wants to create a CLAUDE.md for scheduled or autonomous agents, choose between /loop vs Desktop Scheduled Tasks vs GitHub Actions schedule triggers, design escalation and notification flows for recurring automated tasks, or set up a cron-based autonomous workflow.
---

# Yoruneko — Schedule Task Designer

Help users design, configure, and deploy autonomous scheduled tasks for Claude Code.

## Quick Method Selection

Ask the user about their requirements, then select:

1. **Need persistence across sessions?**
   - No → **`/loop`** (session-scoped, max 3 days, interval-based)
   - Yes → Continue
2. **Machine always on? macOS or Windows?**
   - Yes to both → **Desktop Scheduled Tasks** (persistent, full MCP/Skills access, catch-up supported)
   - No → **GitHub Actions schedule trigger** (cloud, 24/7, no PC needed)
3. **If GitHub Actions — need to modify app code?**
   - Yes → **Pattern A**: Add workflow to the app's own repository
   - No → **Pattern B**: Create a dedicated trigger repository

For detailed comparison, read `patterns/DECISION_MATRIX.md`.

## Design Workflow

Follow these steps to design a complete scheduled task:

### Step 1: Understand the Task

Ask the user:
- What does this task do? (one sentence)
- How often should it run?
- What data does it read? What actions does it take?
- What should happen when something goes wrong?
- Who should be notified of results?

### Step 2: Select Method

Use the Quick Method Selection above. Confirm the choice with the user. Key constraints:
- Desktop Scheduled Tasks: **not available on Linux**
- GitHub Actions: minimum 5-minute interval, UTC cron, may delay on inactive repos
- `/loop`: dies with the session, max 3 days

### Step 3: Design the CLAUDE.md

Every scheduled task CLAUDE.md needs these 7 sections:

1. **Role** — Who the agent is and what it does (one sentence)
2. **Scope** — Allowed and forbidden actions (explicit lists)
3. **Judgment Criteria** — Decision rules with concrete thresholds
4. **Escalation Conditions** — When to stop and ask humans (see Step 4)
5. **Output Format** — How to present results (PR format, notifications, reports)
6. **External Integrations** — APIs, webhooks, auth (environment variables, never hardcode)
7. **Guardrails** — Hard limits (max files, timeout, rollback procedures)

Select and customize a template based on the use case:

| Use Case | Template |
|---|---|
| Customer support triage | `templates/claude-md/cs-automation.md` |
| Automated code review | `templates/claude-md/code-review.md` |
| Infrastructure monitoring | `templates/claude-md/monitoring.md` |
| Daily/weekly reports | `templates/claude-md/reporting.md` |
| Other / custom | `templates/claude-md/custom.md` |

Read the selected template file and adapt it to the user's specific requirements.

### Step 4: Design Escalation

Read `patterns/ESCALATION.md` for the full guide. Key principle: **define what the agent cannot do before defining what it can do.**

Minimum escalation design:
- What triggers a stop (agent must not proceed)
- Where to report (GitHub issue, Slack channel, PagerDuty)
- What context to include in the escalation message

### Step 5: Design Notifications

Read `patterns/NOTIFICATION.md` for integration patterns. Every task needs:
- Success notification (can be daily digest)
- Failure notification (immediate)
- Escalation notification (immediate, different channel from success)

### Step 6: Set Up the Trigger

**For `/loop`:**
```
/loop {interval} {task description referencing CLAUDE.md}
```

**For Desktop Scheduled Tasks:**
Guide the user through the Desktop app's schedule configuration UI.

**For GitHub Actions:**
Read `templates/github-actions/schedule-trigger.yml` and customize:
- Cron schedule
- Required secrets
- Tool permissions
- Timeout

If the task needs an API endpoint, read the appropriate template:
- `templates/api/nextjs-vercel.md` for Next.js + Vercel
- `templates/api/generic-webhook.md` for other setups

### Step 7: Review Against Anti-Patterns

Read `patterns/ANTI_PATTERNS.md` and verify the design against the checklist:
- [ ] Escalation conditions defined
- [ ] Notifications configured (success + failure)
- [ ] Uses PRs, not direct commits to main
- [ ] CLAUDE.md is focused (under ~4,000 tokens)
- [ ] Scope explicitly limited
- [ ] No schedule overlap with other tasks
- [ ] Target OS supports chosen method
- [ ] Error handling is specific

Present the final design to the user for review before implementation.
