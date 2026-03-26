# API Pattern: Generic Webhook

Pattern for creating a generic webhook endpoint that scheduled Claude Code tasks can call, independent of framework or hosting.

## Architecture

```
External Cron / Schedule → Webhook Endpoint → Process + Return Data
Claude Code Agent → Webhook Endpoint → Read/Write Application Data
```

The webhook serves as the bridge between your scheduled Claude Code agent and your application data.

## Implementation Examples

### Express.js (Node.js)

```javascript
const express = require("express");
const crypto = require("crypto");
const app = express();

app.use(express.json());

// HMAC signature verification middleware
function verifySignature(req, res, next) {
  const signature = req.headers["x-webhook-signature"];
  const payload = JSON.stringify(req.body);
  const expected = crypto
    .createHmac("sha256", process.env.WEBHOOK_SECRET)
    .update(payload)
    .digest("hex");

  if (signature !== expected) {
    return res.status(401).json({ error: "Invalid signature" });
  }
  next();
}

// GET: Agent reads data
app.get("/webhook/agent", (req, res) => {
  const token = req.headers["authorization"];
  if (token !== `Bearer ${process.env.WEBHOOK_SECRET}`) {
    return res.status(401).json({ error: "Unauthorized" });
  }

  // Return data for the agent to process
  const data = getDataForAgent();
  res.json({ success: true, data });
});

// POST: Agent writes results
app.post("/webhook/agent", verifySignature, (req, res) => {
  const { action, payload } = req.body;
  const result = processAgentAction(action, payload);
  res.json({ success: true, result });
});

app.listen(process.env.PORT || 3000);
```

### Flask (Python)

```python
import hmac
import hashlib
import os
from flask import Flask, request, jsonify

app = Flask(__name__)

def verify_signature(payload, signature):
    expected = hmac.new(
        os.environ["WEBHOOK_SECRET"].encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(signature, expected)

@app.route("/webhook/agent", methods=["GET"])
def get_data():
    token = request.headers.get("Authorization")
    if token != f"Bearer {os.environ['WEBHOOK_SECRET']}":
        return jsonify({"error": "Unauthorized"}), 401

    data = get_data_for_agent()
    return jsonify({"success": True, "data": data})

@app.route("/webhook/agent", methods=["POST"])
def process_action():
    signature = request.headers.get("X-Webhook-Signature", "")
    if not verify_signature(request.data, signature):
        return jsonify({"error": "Invalid signature"}), 401

    body = request.json
    result = process_agent_action(body["action"], body["payload"])
    return jsonify({"success": True, "result": result})
```

## HMAC Signature Verification

Always verify webhook payloads using HMAC-SHA256:

```
Signature = HMAC-SHA256(secret, request_body)
Header: X-Webhook-Signature: {signature}
```

The agent should sign outgoing POST requests:

```bash
# Generate signature for a payload
echo -n '{"action":"update","payload":{}}' | \
  openssl dgst -sha256 -hmac "$WEBHOOK_SECRET" | \
  awk '{print $2}'
```

## Request/Response Format

### GET (Read data)
```
GET /webhook/agent
Authorization: Bearer {secret}

Response:
{
  "success": true,
  "data": {
    "items": [...],
    "metadata": {
      "total": 42,
      "since": "2025-01-01T00:00:00Z"
    }
  }
}
```

### POST (Write results)
```
POST /webhook/agent
Content-Type: application/json
X-Webhook-Signature: {hmac_signature}

Body:
{
  "action": "update_ticket",
  "payload": {
    "ticket_id": "123",
    "status": "resolved",
    "response_draft": "..."
  }
}

Response:
{
  "success": true,
  "result": {
    "updated": true,
    "id": "123"
  }
}
```

## External Cron Services

If your hosting doesn't support built-in cron, use an external service to trigger the endpoint:

- **cron-job.org** — Free, reliable, supports HTTPS
- **EasyCron** — Free tier available, HTTP header support
- **GitHub Actions** — Use `schedule` trigger with `curl` step
- **Uptime monitors** — Some (e.g., UptimeRobot) can serve as basic cron

## CLAUDE.md Integration

```markdown
# External Integrations

## Data Source
- Read data: GET $WEBHOOK_URL/webhook/agent
  Headers: Authorization: Bearer $WEBHOOK_SECRET

## Actions
- Write results: POST $WEBHOOK_URL/webhook/agent
  Headers:
    Content-Type: application/json
    X-Webhook-Signature: HMAC-SHA256($WEBHOOK_SECRET, request_body)
  Body: { "action": "...", "payload": {...} }
```

## Security Checklist

- [ ] HMAC signature verification on all POST requests
- [ ] Bearer token authentication on all GET requests
- [ ] Secret stored in environment variable, never in code
- [ ] HTTPS only (no plain HTTP)
- [ ] Rate limiting configured
- [ ] Request logging for audit trail
- [ ] Input validation on all incoming payloads
