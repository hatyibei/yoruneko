---
name: yoruneko
description: Design and set up Claude Code scheduled tasks (/loop, Desktop Scheduled Tasks, GitHub Actions schedule trigger). Guides method selection, CLAUDE.md template design, escalation rules, and notification setup for autonomous recurring tasks. Use when setting up a scheduled agent, designing a CLAUDE.md for automation, choosing between schedule methods, planning escalation and monitoring for autonomous tasks, or asking how to automate recurring work with Claude Code.
---

# Yoruneko — Schedule Task Designer

Help users design, configure, and deploy autonomous scheduled tasks for Claude Code.

## Quick Method Selection

Ask the user about their requirements, then select:

1. **Need persistence across sessions?**
   - No → **`/loop`** (session-scoped, max 3 days, interval-based). Best for transient monitoring like watching a deploy or CI status — tasks that naturally end when you close the terminal.
   - Yes → Continue
2. **Machine always on? macOS or Windows?**
   - Yes to both → **Desktop Scheduled Tasks** (persistent, full MCP/Skills access, catch-up supported). Best when tasks need local tools, MCP servers, or files only available on your machine.
   - No → **GitHub Actions schedule trigger** (cloud, 24/7, no PC needed). Best for tasks requiring server-grade reliability — they run even when your machine is off.
3. **If GitHub Actions — need to modify app code?**
   - Yes → **Pattern A**: Add workflow to the app's own repository. Simpler setup; the agent can directly commit and create PRs.
   - No → **Pattern B**: Create a dedicated trigger repository. Keeps automation separate from app code; the agent calls external APIs instead.

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

Every scheduled task CLAUDE.md needs these 7 sections. Each exists for a specific reason — without any one of them, the autonomous agent will make avoidable mistakes:

1. **Role** — Who the agent is and what it does (one sentence). Why: Without a clear role, the agent's tone and decision-making are inconsistent across runs.
2. **Scope** — Allowed and forbidden actions (explicit lists). Why: Autonomous agents have no human watching each action. Explicit boundaries prevent scope creep.
3. **Judgment Criteria** — Decision rules with concrete thresholds. Why: Vague criteria ("if it seems important") produce inconsistent behavior. Concrete thresholds ("error rate > 5%") make the agent predictable.
4. **Escalation Conditions** — When to stop and ask humans (see Step 4). Why: The costliest failure mode is an agent that confidently does the wrong thing. Escalation is cheaper than cleanup.
5. **Output Format** — How to present results (PR format, notifications, reports). Why: Scheduled tasks run unattended. Consistent output makes results scannable at a glance.
6. **External Integrations** — APIs, webhooks, auth (environment variables, never hardcode). Why: Hardcoded secrets in CLAUDE.md will be committed to version control. Environment variables keep secrets separate.
7. **Guardrails** — Hard limits (max files, timeout, rollback procedures). Why: Without hard limits, a single bad run can modify hundreds of files or run indefinitely.

Select and customize a template based on the use case:

| Use Case | Template |
|---|---|
| Customer support triage | `templates/claude-md/cs-automation.md` |
| Automated code review | `templates/claude-md/code-review.md` |
| Infrastructure monitoring | `templates/claude-md/monitoring.md` |
| Daily/weekly reports | `templates/claude-md/reporting.md` |
| Other / custom | `templates/claude-md/custom.md` |

For CLI/Desktop installations, read the selected template file and adapt it to the user's specific requirements.

For browser-only installations (Claude.ai Skills UI), templates are not available as separate files. In that case, use the 7-section structure above as a framework and build the CLAUDE.md from scratch, following the WHY guidance for each section.

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
