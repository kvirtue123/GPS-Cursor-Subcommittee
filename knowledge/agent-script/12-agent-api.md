# Agent API — REST API for Agentforce Agents

> **Sources:**  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api.html  
> - https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-get-started.html  
> **Scraped:** 2026-04-20

---

## What is the Agent API?

The **Agent API** is a REST API that lets you communicate with Agentforce agents from anywhere that can call a REST API. It provides access to your goal-oriented, autonomous AI agents.

> Note: Agent API is **not supported** for agents of type "Agentforce (Default)".

---

## Use Cases

- Connect to an Agentforce agent from a **website**
- Create and deploy **headless agents** to automate functionality without UI constraints
- Connect agents to **platforms and workflows** using standardized API endpoints
- Build an **agentic ecosystem** by invoking Agentforce agents from other agents

---

## Prerequisites

1. Agentforce must be **enabled** with at least one **activated** agent
2. An **External Client App (ECA)** set up with the client credentials flow

---

## Setup: Create an External Client App

1. From Setup, find **External Client Apps Manager**
2. Click **New External Client App**
3. Specify a name and contact email
4. Select **Enable OAuth** with these scopes:
   - `api` — Manage user data via APIs
   - `refresh_token, offline_access` — Perform requests at any time
   - `chatbot_api` — Access chatbot services
   - `sfap_api` — Access the Salesforce API Platform
5. Enable additional OAuth settings:
   - ✅ **Enable Client Credentials Flow**
   - ✅ **Issue JWT Web Token (JWT)-based access tokens for named users**
6. Deselect:
   - ❌ Require secret for Web Server Flow
   - ❌ Require secret for Refresh Token Flow
   - ❌ Require Proof Key for Code Exchange (PKCE)
7. Save the app
8. Go to the **Policy** tab → Edit → Under **OAuth Flows and External Client App Enhancements**:
   - ✅ Enable Client Credentials Flow
   - Set **Run As (Username)** to a user with at least API Only access

---

## Obtain Credentials

From your External Client App:
1. Click **Settings** tab
2. Expand **OAuth Settings**
3. Click **Consumer Key and Secret** → Copy your key and secret

---

## Step 1: Get an Access Token

Use your Consumer Key, Consumer Secret, and domain URL to mint a token:

```bash
curl -X POST "https://MY_DOMAIN_URL/services/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=CONSUMER_KEY" \
  -d "client_secret=CONSUMER_SECRET"
```

Response:
```json
{
  "access_token": "00D...",
  "instance_url": "https://your-domain.my.salesforce.com",
  "id": "https://login.salesforce.com/id/...",
  "token_type": "Bearer",
  "issued_at": "1713600000000",
  "signature": "..."
}
```

Copy the `access_token` — required for all subsequent API calls.

> **Important**: Use your **My Domain URL** (e.g., `some_domain.my.salesforce.com`), NOT `some_domain.lightning.force.com`.

---

## Step 2: Get the Agent ID

Get the Agent ID of the agent you want to interact with from Setup or using the Tooling API. Refer to: [Get the Agent ID for an Agent](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-agent-id.html)

---

## Step 3: Start a Session

```bash
curl -X POST "https://MY_DOMAIN_URL/einstein/ai-agent/v1/agents/AGENT_ID/sessions" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "externalSessionKey": "RANDOM_UUID",
    "instanceConfig": {
      "endpoint": "https://MY_DOMAIN_URL"
    },
    "bypassUser": true
  }'
```

The `bypassUser` parameter:
- `true` — Uses the agent-assigned user instead of the logged-in user (recommended for client credentials flow)
- `false` — Uses the user associated with the token

**Response** includes a `sessionId` — use this to continue the conversation.

---

## Step 4: Continue the Conversation

Send messages (utterances) using the session ID:

```bash
curl -X POST "https://MY_DOMAIN_URL/einstein/ai-agent/v1/sessions/SESSION_ID/messages" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What is the status of my order?"
        }
      ]
    },
    "variables": []
  }'
```

---

## Session Lifecycle

1. **Create session** — `POST /einstein/ai-agent/v1/agents/{agentId}/sessions`
2. **Send messages** — `POST /einstein/ai-agent/v1/sessions/{sessionId}/messages`
3. **End session** — `DELETE /einstein/ai-agent/v1/sessions/{sessionId}`

See [Agent API Session Lifecycle](https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-lifecycle.html) for full details.

---

## Connection to Agentforce DX Preview

The **Agentforce DX preview functionality** (VS Code Agent Preview pane and `agent preview` CLI command) uses the **Agent API internally**. The transcript/trace files saved by the preview contain the raw API messages, giving you insight into:
- How the agent interprets utterances
- Which actions the agent selects and why
- API calls and data transformations
- Response generation and output formatting

---

## Additional API Resources

| Resource | Link |
|----------|------|
| Agent API Developer Guide | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api.html |
| Get Started with Agent API | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-get-started.html |
| Agent API Session Lifecycle | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-lifecycle.html |
| Agent API Examples | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-examples.html |
| Agent API Considerations | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-considerations.html |
| Agent API Troubleshooting | https://developer.salesforce.com/docs/ai/agentforce/guide/agent-api-troubleshooting.html |
| Agent API Reference | https://developer.salesforce.com/docs/ai/agentforce/references/agent-api |
| Agent API Postman Collection | https://www.postman.com/salesforce-developers/salesforce-developers/collection/gwv9bjy/agent-api |
| Calling an Agent from Flow or Apex | https://help.salesforce.com/s/articleView?id=ai.agent_custom_invocable_action_flow_apex.htm |
| Testing API | https://developer.salesforce.com/docs/ai/agentforce/guide/testing-api.html |
