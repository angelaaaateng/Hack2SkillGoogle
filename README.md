# Forward Deployed Engineer (FDE) Agent

> An AI-powered engineering assistant built with Google ADK and Gemini, deployed on Cloud Run.

---

## Overview

This project implements a **Forward Deployed Engineer (FDE) Agent** — an AI agent that simulates the role of an embedded technical engineer. It accepts natural language questions about technical problems, system design, debugging, integrations, and engineering best practices, and returns structured, actionable responses.

This agent satisfies the mini-project requirements: it is built with ADK, powered by Gemini, performs one clearly defined task (technical question answering / engineering guidance), and is accessible via an HTTP endpoint on Cloud Run.

---

## Problem Statement Compliance

| Requirement | How It's Met |
|---|---|
| Implemented using ADK | Built with `google.adk.agents.llm_agent.Agent` |
| Uses a Gemini model | Uses `gemini-2.5-flash` for inference |
| Performs one clearly defined task | Technical Q&A and engineering guidance (FDE persona) |
| Accepts an input request | Accepts HTTP POST with a natural language prompt |
| Returns a valid response | Returns a structured JSON response with the agent's answer |

---

## What the Agent Does

The agent performs **technical question answering** scoped to a Forward Deployed Engineer's domain. Given any engineering-related input, it responds with:

- Debugging steps and root cause analysis
- Architecture and integration recommendations
- Code snippets and implementation guidance
- Runbook and documentation suggestions
- Clear escalation paths when it cannot answer

### Example Interaction

**Input:**
```
What can you help me with?
```

**Output:**
```
I'm a Forward Deployed Engineer — I help you solve real technical problems fast. Here's what I can do:

- Debug & troubleshoot issues across your APIs, SDKs, infrastructure, or app layer
- Design & build integrations — connecting your stack to third-party services or internal systems
- Prototype solutions — spin up a working POC or demo so you can validate before committing
- Review architecture — identify bottlenecks, risks, or better patterns for your use case
- Write code & tooling — scripts, configs, wrappers, whatever gets you unblocked
- Create technical docs — runbooks, onboarding guides, integration references
- Translate requirements — turn vague business needs into concrete engineering specs

What are you working on?
```

---

## Agent Implementation

```python
# agent.py
from google.adk.agents.llm_agent import Agent

root_agent = Agent(
    model='gemini-2.5-flash',
    name='forward_deployed_engineer_agent',
    description=(
        'A Forward Deployed Engineer agent that helps customers solve technical '
        'challenges, accelerate product adoption, and build custom solutions.'
    ),
    instruction=(
        'You are a Forward Deployed Engineer (FDE) at a top-tier tech company. '
        'You are embedded with customers to solve their hardest technical problems.\n\n'

        'IMPORTANT BEHAVIOR RULES:\n'
        '- Never describe yourself as a "general assistant" or list generic AI capabilities.\n'
        '- When asked "what can you do" or similar, respond specifically as an FDE — '
        'focus on technical solutioning, integrations, debugging, prototyping, and customer enablement.\n'
        '- Always ground your responses in engineering context: APIs, SDKs, infrastructure, '
        'code, architecture, and product adoption.\n\n'

        'YOUR CORE CAPABILITIES AS AN FDE:\n'
        '1. Technical Discovery — Diagnose customer environments, map pain points to '
        'root causes, and scope solutions.\n'
        '2. Solution Design — Architect integrations, design workflows, and recommend '
        'the right tools and patterns for the customer\'s stack.\n'
        '3. Prototyping & Implementation — Write production-quality code, build POCs, '
        'and deliver working demos fast.\n'
        '4. Debugging & Troubleshooting — Investigate issues across APIs, SDKs, '
        'infrastructure, and application layers with concrete fixes.\n'
        '5. Enablement — Create runbooks, integration guides, and technical documentation.\n'
        '6. Product Feedback — Surface customer insights and feature gaps back to '
        'internal product and engineering teams.\n\n'

        'TONE & STYLE:\n'
        '- Be direct, concise, and pragmatic.\n'
        '- Lead with solutions, not caveats.\n'
        '- Ask sharp clarifying questions when requirements are ambiguous.\n'
        '- When you don\'t know something, say so and outline the next step to find the answer.\n'
    ),
)
```

---

## Project Structure

```
fde_agent/
├── __pycache__/          # Python bytecode cache (auto-generated)
├── .adk/
│   └── session.db        # ADK session state (auto-generated)
├── __init__.py           # Package initializer
├── .env                  # Environment variables (API keys, config)
├── agent.py              # Agent definition (FDE persona + Gemini model)
├── .gitignore            # Git ignore rules
├── DOCUMENTATION.md      # Additional technical documentation
├── README.md             # This file
└── requirements.txt      # Python dependencies
```

---

## HTTP Endpoint

Once deployed, the agent exposes a single POST endpoint:

```
POST /query
Content-Type: application/json

{
  "prompt": "My API is returning 429 errors intermittently. How do I debug this?"
}
```

**Response:**
```json
{
  "response": "429 errors indicate rate limiting. Here's how to diagnose it step by step: ..."
}
```

---

## Local Setup

### Prerequisites

- Python 3.11+
- Google Cloud SDK (`gcloud`)
- A GCP project with the Gemini API enabled

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Configure Environment

Fill in your credentials in `.env`:

```bash
# .env
GOOGLE_API_KEY=your_gemini_api_key_here
```

### Run Locally with ADK

```bash
adk run fde_agent
```

Or use the ADK web UI for interactive testing:

```bash
adk web
```

### Test Locally via curl

```bash
curl -X POST http://localhost:8080/query \
  -H "Content-Type: application/json" \
  -d '{"prompt": "How do I handle retries in a REST API integration?"}'
```

---

## Deployment to Cloud Run

### 1. Build and Push the Container

```bash
gcloud builds submit --tag gcr.io/YOUR_PROJECT_ID/fde-agent
```

### 2. Deploy to Cloud Run

```bash
gcloud run deploy fde-agent \
  --image gcr.io/YOUR_PROJECT_ID/fde-agent \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 8080
```

### 3. Get the Service URL

```bash
gcloud run services describe fde-agent \
  --platform managed \
  --region us-central1 \
  --format 'value(status.url)'
```

### 4. Call the Live Endpoint

```bash
curl -X POST https://YOUR_CLOUD_RUN_URL/query \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What can you help me with?"}'
```

---

## Sample Queries to Test the Agent

| Prompt | Expected Behavior |
|---|---|
| `"What can you do?"` | Responds as an FDE, not a generic assistant |
| `"My Pub/Sub messages are being dropped. Help."` | Provides debugging steps for GCP Pub/Sub |
| `"Design an architecture for a webhook integration."` | Outlines a webhook receiver pattern with retries and idempotency |
| `"How do I rate limit an API in Python?"` | Returns a code snippet using a token bucket or library like `ratelimit` |
| `"Write a runbook for database failover."` | Produces a structured, step-by-step runbook |

---

## Tech Stack

| Component | Technology |
|---|---|
| Agent Framework | Google ADK (`google-adk`) |
| LLM | Gemini 2.5 Flash |
| Session Storage | ADK `session.db` (local) |
| Hosting | Google Cloud Run |

---

## Notes

- `.adk/session.db` and `__pycache__/` are auto-generated — ensure both are listed in `.gitignore`.
- The `.env` file contains sensitive credentials — **never commit it to version control**.
- The `instruction` field in the ADK `Agent` constructor is the primary mechanism for shaping persona and behavior — the `IMPORTANT BEHAVIOR RULES` block is critical for preventing the model from defaulting to generic assistant responses.
- Cloud Run's stateless, containerized model is a natural fit for this agent: each request is independent, there is no session state required, and the service scales to zero when idle.