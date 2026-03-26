# Anti-Patterns

Things that go wrong when designing scheduled autonomous tasks. Avoid these.

---

## 1. Context Explosion

**Problem**: Stuffing everything into a single CLAUDE.md until it exceeds the context window.

**Symptoms**: Agent ignores instructions at the bottom, behaves inconsistently, slow response times.

**Fix**: Keep CLAUDE.md focused on decision rules and boundaries. Move reference data (FAQ databases, code style guides, API docs) into separate files that the agent reads as needed.

```markdown
# BAD: 200-line FAQ pasted inline
## FAQ
Q: How do I reset my password? A: Go to settings...
Q: How do I change my email? A: Go to profile...
(... 150 more entries ...)

# GOOD: Reference to external file
When answering customer questions, check docs/faq.md first.
If the answer isn't there, check docs/troubleshooting.md.
If neither helps, escalate.
```

---

## 2. No Escalation Design

**Problem**: Autonomous agent with no defined boundary for "things I shouldn't handle."

**Symptoms**: Agent makes incorrect decisions on edge cases. Refunds issued without approval. Wrong answers sent to customers.

**Fix**: Define explicit escalation conditions. See `patterns/ESCALATION.md`.

**Rule of thumb**: If a wrong decision costs more than 5 minutes of human time to fix, it needs an escalation condition.

---

## 3. Direct Commit to Main

**Problem**: Scheduled agent pushes directly to the main/production branch without review.

**Symptoms**: Broken production deployments. Untested code in production. No audit trail.

**Fix**: Always use pull requests, even for automated changes.

```markdown
# BAD
Push your changes directly to the main branch.

# GOOD
Create a new branch named auto/{date}-{description}.
Open a pull request targeting main.
Add the "automated" label.
Do NOT merge - a human will review and merge.
```

---

## 4. No Notification Design

**Problem**: Scheduled agent runs silently. No one knows if it's working, broken, or stuck.

**Symptoms**: Agent has been failing for days before anyone notices. Duplicate work because humans don't know the agent already handled something.

**Fix**: Every scheduled task must notify on both success and failure. See `patterns/NOTIFICATION.md`.

**Minimum**: Log the result somewhere visible. Ideal: structured notifications per the notification guide.

---

## 5. Multiple Schedules on Same Cycle

**Problem**: Setting up multiple independent scheduled tasks that run at the same time and interfere with each other.

**Symptoms**: Race conditions. Conflicting git operations. Duplicate PRs. Resource contention.

**Fix**: Stagger schedules by at least 15 minutes, or combine related tasks into a single scheduled workflow.

```yaml
# BAD: Both at midnight
- cron: '0 0 * * *'  # code-review
- cron: '0 0 * * *'  # dependency-update

# GOOD: Staggered
- cron: '0 0 * * *'  # code-review at midnight
- cron: '0 1 * * *'  # dependency-update at 1 AM
```

---

## 6. Desktop Scheduled Tasks on Linux

**Problem**: Attempting to use Desktop Scheduled Tasks on a Linux machine.

**Symptoms**: Configuration appears to save but tasks never execute. No error message.

**Fix**: Use GitHub Actions schedule trigger for Linux environments. Desktop Scheduled Tasks only support macOS and Windows.

---

## 7. Missing Guardrails

**Problem**: Agent has broader permissions than it needs. No limits on what files it can touch or what actions it can take.

**Symptoms**: Agent modifies unrelated files. Agent runs destructive commands. Scope creep over time.

**Fix**: Define explicit scope boundaries.

```markdown
# BAD
You are a code review agent. Review PRs and suggest fixes.

# GOOD
You are a code review agent.
- ONLY review PRs with the "needs-review" label
- ONLY comment on files matching src/**/*.ts
- NEVER approve or merge PRs
- NEVER modify code directly
- If a PR touches packages/ or .github/, skip it and notify #dev-ops
```

---

## 8. Overly Broad Error Handling

**Problem**: Agent catches all errors with a generic handler instead of addressing specific failure modes.

**Symptoms**: Real errors get swallowed. Agent reports "completed successfully" when it actually failed silently.

**Fix**: Handle known error types explicitly. Let unknown errors escalate.

```markdown
# BAD
If any error occurs, log it and continue.

# GOOD
If the API returns 429 (rate limit), wait 60 seconds and retry once.
If the API returns 401 (auth), stop and notify #ops-alerts.
If the API returns 5xx, retry up to 3 times with 30s delay.
For any other error, stop and create a GitHub issue with the full error.
```

---

## Quick Checklist

Before deploying a scheduled task, verify:

- [ ] Escalation conditions are defined for edge cases
- [ ] Notification is configured for both success and failure
- [ ] Agent uses PRs, not direct commits to main
- [ ] CLAUDE.md is under a reasonable size (aim for under 4,000 tokens)
- [ ] File and action scope is explicitly limited
- [ ] Schedule doesn't overlap with other automated tasks
- [ ] Target OS supports the chosen schedule method
- [ ] Error handling is specific, not generic catch-all
