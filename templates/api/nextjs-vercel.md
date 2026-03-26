# API Pattern: Next.js + Vercel

Pattern for creating a thin API endpoint that serves as a trigger for scheduled Claude Code tasks, deployed on Vercel.

## Architecture

```
Vercel Cron → /api/schedule-trigger → Fetch data → Return to Claude Code
```

The API endpoint doesn't run Claude Code itself. It serves as the "eyes and hands" that the scheduled Claude Code agent calls to read and write data.

## API Route Implementation

### `app/api/schedule-trigger/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  // Verify cron secret to prevent unauthorized access
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Your data fetching logic here
  // Example: fetch support tickets, metrics, or any data the agent needs
  try {
    const data = await fetchDataForAgent();
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to fetch data" },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Handle agent's write operations
  // Example: send a drafted response, update a ticket status
  const body = await request.json();

  try {
    const result = await processAgentAction(body);
    return NextResponse.json({ success: true, result });
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to process action" },
      { status: 500 }
    );
  }
}
```

## Vercel Cron Configuration

### `vercel.json`

```json
{
  "crons": [
    {
      "path": "/api/schedule-trigger",
      "schedule": "0 * * * *"
    }
  ]
}
```

Common schedules:
- `0 * * * *` — Every hour
- `0 9 * * *` — Daily at 9:00 UTC
- `0 9 * * 1-5` — Weekdays at 9:00 UTC
- `*/15 * * * *` — Every 15 minutes

## Environment Variables

Set these in Vercel Dashboard > Settings > Environment Variables:

| Variable | Purpose |
|---|---|
| `CRON_SECRET` | Auth token for cron endpoint |
| Any app-specific vars | Database URLs, API keys, etc. |

Generate a secure `CRON_SECRET`:
```bash
openssl rand -base64 32
```

## CLAUDE.md Integration

Reference the API endpoint in your scheduled task's CLAUDE.md:

```markdown
# External Integrations

## Data Source
- Fetch pending items: GET $APP_URL/api/schedule-trigger
  Headers: Authorization: Bearer $CRON_SECRET
  Response: { "success": true, "data": [...] }

## Write Back
- Submit results: POST $APP_URL/api/schedule-trigger
  Headers: Authorization: Bearer $CRON_SECRET
  Body: { "action": "...", "payload": {...} }
```

## Multiple Endpoints Pattern

For complex tasks, create separate endpoints:

```
app/api/schedule/
├── tickets/route.ts     # GET: fetch tickets, POST: update ticket
├── metrics/route.ts     # GET: fetch metrics
└── notify/route.ts      # POST: send notifications
```

## Security Notes

- Always validate `CRON_SECRET` on every request
- Use environment variables for all secrets (never hardcode)
- Vercel cron requests come from Vercel's infrastructure — the `CRON_SECRET` header prevents external access
- Rate limit the endpoints if exposed to the internet
- Log all requests for audit trail
