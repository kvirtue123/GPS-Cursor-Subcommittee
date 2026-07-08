---
title: Agent API - Headless Communication
description: Control your agents programmatically via REST APIs
feature_area: agentforce-agent-script
status: duplicate-of-catalog
catalog_id: P1
catalog_ref: patterns/canonical/agent-script-library.md
note: "Already cataloged as P1 (Agent Script knowledge library). Kept here for local browsing — do not re-catalog."
---

# Agent API - Headless Communication

Interact with your deployed Agentforce agents programmatically using REST APIs. Perfect for building custom integrations, testing workflows, or automating agent interactions.

{% agent-api-interactive /%}

## API Usage Guide

Follow these steps to interact with your agent programmatically:

### Step 0: Get OAuth Access Token

First, get an OAuth access token using your credentials. Copy the `access_token` from the response.

```bash
curl -X POST "{{INSTANCE_URL}}/services/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id={{CLIENT_ID}}" \
  -d "client_secret={{CLIENT_SECRET}}"
```

**Response example:**

```json
{
  "access_token": "00D5e0000...",
  "instance_url": "https://your-org.my.salesforce.com",
  "id": "https://login.salesforce.com/id/...",
  "token_type": "Bearer",
  "issued_at": "1234567890",
  "signature": "..."
}
```

### Step 1: Start Agent Session

Start a new agent session. Save the `sessionId` from the response.

```bash
curl -X POST "{{AGENT_API_BASE_URL}}/agents/{{AGENT_ID}}/sessions" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "externalSessionKey": "session_1234567890",
    "instanceConfig": {
      "endpoint": "{{INSTANCE_URL}}"
    },
    "tz": "America/Los_Angeles",
    "variables": [{
      "name": "$Context.EndUserLanguage",
      "type": "Text",
      "value": "en_US"
    }],
    "bypassUser": true
  }'
```

**Key parameters:**

- `externalSessionKey`: A unique identifier for this session (use timestamps or UUIDs)
- `instanceConfig.endpoint`: Your Salesforce instance URL
- `tz`: Timezone for the session
- `variables`: Context variables for the agent
- `bypassUser`: Set to `true` for headless API access

**Response example:**

```json
{
  "sessionId": "0Xx5e0000...",
  "createdDate": "2024-01-01T00:00:00.000Z",
  "status": "ACTIVE"
}
```

### Step 2: Send Message to Agent

Send a message to the agent. Increment `sequenceId` for each new message (1, 2, 3, ...).

```bash
curl -X POST "{{AGENT_API_BASE_URL}}/sessions/YOUR_SESSION_ID/messages" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "sequenceId": 1,
      "type": "Text",
      "text": "Hello! Can you help me?"
    }
  }'
```

**Important:** Track the `sequenceId` yourself - it must increment with each message in the session.

**Response example:**

```json
{
  "messages": [
    {
      "sequenceId": 1,
      "type": "Text",
      "text": "Hello! I'm here to help. What can I assist you with today?",
      "role": "Agent"
    }
  ]
}
```

### Step 3: End Agent Session

Close the session when done to free up resources.

```bash
curl -X DELETE "{{AGENT_API_BASE_URL}}/sessions/YOUR_SESSION_ID" \
  -H "Authorization: Bearer {{ACCESS_TOKEN}}" \
  -H "x-session-end-reason: UserRequest"
```

**Response:** Returns HTTP 200 with a `SessionEnded` message.

**Note:** The `x-session-end-reason` header is required. Valid values: `UserRequest`, `Transfer`.

## Streaming Responses

For real-time streaming responses, add streaming capabilities to your session:

```json
{
  "featureSupport": "Streaming",
  "streamingCapabilities": {
    "chunkTypes": ["Text"]
  }
}
```

When streaming is enabled, responses will be sent as Server-Sent Events (SSE).

## Error Handling

Common error responses:

| Status Code | Description           | Solution                                     |
| ----------- | --------------------- | -------------------------------------------- |
| 401         | Unauthorized          | OAuth token expired - get a new one          |
| 404         | Not Found             | Check agent ID or session ID                 |
| 429         | Too Many Requests     | Rate limit exceeded - slow down requests     |
| 500         | Internal Server Error | Salesforce server issue - retry with backoff |

## Rate Limits

- **Sessions per hour**: 1000 per org
- **Messages per session**: 100
- **Concurrent sessions**: 50 per org

## SDK Examples

{% code-variants %}
{% variant label="JavaScript/Node.js" language="javascript" %}

```javascript
const axios = require("axios");

const client = new AgentAPIClient(
  "{{INSTANCE_URL}}",
  "{{CLIENT_ID}}",
  "{{CLIENT_SECRET}}"
);

class AgentAPIClient {
  constructor(instanceUrl, clientId, clientSecret) {
    this.instanceUrl = instanceUrl;
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.baseUrl = "https://api.salesforce.com/einstein/ai-agent/v1";
  }

  async getAccessToken() {
    const response = await axios.post(
      `${this.instanceUrl}/services/oauth2/token`,
      new URLSearchParams({
        grant_type: "client_credentials",
        client_id: this.clientId,
        client_secret: this.clientSecret,
      }),
      { headers: { "Content-Type": "application/x-www-form-urlencoded" } }
    );
    return response.data.access_token;
  }

  async startSession(agentId, accessToken) {
    const response = await axios.post(
      `${this.baseUrl}/agents/${agentId}/sessions`,
      {
        externalSessionKey: `session_${Date.now()}`,
        instanceConfig: { endpoint: this.instanceUrl },
        tz: "America/Los_Angeles",
        bypassUser: true,
      },
      { headers: { Authorization: `Bearer ${accessToken}` } }
    );
    return response.data.sessionId;
  }

  async sendMessage(sessionId, text, sequenceId, accessToken) {
    const response = await axios.post(
      `${this.baseUrl}/sessions/${sessionId}/messages`,
      {
        message: {
          sequenceId,
          type: "Text",
          text,
        },
      },
      { headers: { Authorization: `Bearer ${accessToken}` } }
    );
    return response.data;
  }

  async endSession(sessionId, accessToken) {
    await axios.delete(`${this.baseUrl}/sessions/${sessionId}`, {
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "x-session-end-reason": "UserRequest",
      },
    });
  }
}

// Usage
(async () => {
  const token = await client.getAccessToken();
  const sessionId = await client.startSession("{{AGENT_ID}}", token);
  const response = await client.sendMessage(sessionId, "Hello!", 1, token);
  console.log(response);
  await client.endSession(sessionId, token);
})();
```

{% /variant %}

{% variant label="Python" language="python" %}

```python
import requests
import time

client = AgentAPIClient(
    "{{INSTANCE_URL}}",
    "{{CLIENT_ID}}",
    "{{CLIENT_SECRET}}"
)

class AgentAPIClient:
    def __init__(self, instance_url, client_id, client_secret):
        self.instance_url = instance_url
        self.client_id = client_id
        self.client_secret = client_secret
        self.base_url = 'https://api.salesforce.com/einstein/ai-agent/v1'

    def get_access_token(self):
        response = requests.post(
            f"{self.instance_url}/services/oauth2/token",
            data={
                'grant_type': 'client_credentials',
                'client_id': self.client_id,
                'client_secret': self.client_secret
            }
        )
        return response.json()['access_token']

    def start_session(self, agent_id, access_token):
        response = requests.post(
            f"{self.base_url}/agents/{agent_id}/sessions",
            json={
                'externalSessionKey': f"session_{int(time.time())}",
                'instanceConfig': {'endpoint': self.instance_url},
                'tz': 'America/Los_Angeles',
                'bypassUser': True
            },
            headers={'Authorization': f'Bearer {access_token}'}
        )
        return response.json()['sessionId']

    def send_message(self, session_id, text, sequence_id, access_token):
        response = requests.post(
            f"{self.base_url}/sessions/{session_id}/messages",
            json={
                'message': {
                    'sequenceId': sequence_id,
                    'type': 'Text',
                    'text': text
                }
            },
            headers={'Authorization': f'Bearer {access_token}'}
        )
        return response.json()

    def end_session(self, session_id, access_token):
        requests.delete(
            f"{self.base_url}/sessions/{session_id}",
            headers={
                'Authorization': f'Bearer {access_token}',
                'x-session-end-reason': 'UserRequest'
            }
        )

# Usage
token = client.get_access_token()
session_id = client.start_session("{{AGENT_ID}}", token)
response = client.send_message(session_id, "Hello!", 1, token)
print(response)
client.end_session(session_id, token)
```

{% /variant %}
{% /code-variants %}

## Tips

- **Token expiration**: OAuth tokens expire after ~2 hours - implement token refresh logic
- **Session management**: Always end sessions when done to avoid hitting Salesforce limits
- **Sequence tracking**: Maintain sequence IDs on your side - they must increment sequentially
- **Error handling**: Implement retry logic with exponential backoff for transient errors
- **Rate limiting**: Respect rate limits and implement proper throttling

## Additional Resources

- [Official Salesforce Agent API Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/agent-api.html)
- [OAuth 2.0 Client Credentials Flow](https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_client_credentials_flow.htm)
- [Einstein Platform Rate Limits](https://developer.salesforce.com/docs/einstein/genai/guide/rate-limits.html)
