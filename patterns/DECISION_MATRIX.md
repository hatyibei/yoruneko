# Schedule Method Decision Matrix

## Comparison Table

| Feature | `/loop` | Desktop Scheduled Tasks | GitHub Actions Schedule |
|---|---|---|---|
| **Persistence** | Session only (max 3 days) | Permanent (survives restart) | Permanent (cloud) |
| **OS Support** | All (CLI) | macOS, Windows | All (cloud) |
| **PC Required** | Yes (running session) | Yes (machine on) | No |
| **Cron Syntax** | Interval only (e.g. `5m`) | Full cron | Full cron (UTC) |
| **Min Interval** | 1 minute | 1 minute | 5 minutes |
| **Cost** | Local compute | Local compute | Free (Pro+) / API credits |
| **MCP/Skills Access** | Full | Full | CLAUDE.md only |
| **Catch-up** | No | Yes (past 7 days, 1x) | No |
| **Setup Complexity** | One command | GUI | Workflow YAML + secrets |
| **Internet Required** | For API calls only | For API calls only | Always (cloud) |

## When to Use Each

### `/loop` - Short-lived Polling

Best for:
- Monitoring a deployment in progress
- Watching CI status until green
- Polling an API during a debug session
- Any task that ends when your session ends

```
/loop 5m check deployment status on staging
```

Key limitation: Dies when you close the terminal or after 3 days max.

### Desktop Scheduled Tasks - Persistent Local

Best for:
- Daily report generation (your machine is always on)
- Periodic code review of new PRs
- Local file processing on a schedule
- Tasks that need MCP servers or local tools

Key limitations:
- **Not available on Linux**
- Machine must be powered on at scheduled time
- Catch-up runs missed tasks (past 7 days) but only once

### GitHub Actions Schedule Trigger - Persistent Cloud

Best for:
- CS ticket triage (24/7, no PC needed)
- Automated issue management
- Cross-repo maintenance tasks
- Anything requiring server-grade reliability

Two patterns:
- **Pattern A (In-repo)**: Add workflow to your app's repo. Use when the scheduled task modifies the same repo's code.
- **Pattern B (Dedicated repo)**: Create a separate trigger repo. Use when the task orchestrates across repos or doesn't need code changes.

Key limitations:
- GitHub may delay or skip runs on inactive repos
- No MCP server access (CLAUDE.md-driven only)
- Minimum 5-minute interval
- UTC timezone for cron expressions

## Decision Flow

```
Need persistence across sessions?
├─ No → /loop
└─ Yes
    ├─ Machine always on? + macOS/Windows?
    │   ├─ Yes → Desktop Scheduled Tasks
    │   └─ No → GitHub Actions schedule trigger
    └─ Need 24/7 reliability regardless of machine state?
        └─ Yes → GitHub Actions schedule trigger
```

## Hybrid Patterns

You can combine methods:

- **GitHub Actions (daily) + /loop (ad-hoc)**: Use Actions for routine tasks, `/loop` for debugging or one-off monitoring
- **Desktop Tasks (business hours) + GitHub Actions (overnight)**: Local processing during work, cloud processing during off-hours
