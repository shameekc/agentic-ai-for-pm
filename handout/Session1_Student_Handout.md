# Session 1 Student Handout — Agent-driven Automation in Products

**Institute of Product Leadership**
**Course:** Agent-driven Automation in Products
**Session 1:** Foundations of Agentic AI and Agent Design Patterns

---

## Session Agenda

| Block | Topic | Duration |
|-------|-------|----------|
| 1 | Google Antigravity PM workflows demo | 25 min |
| 2 | Agentic AI foundations + 8 design patterns overview | 30 min |
| 3 | Make.com build-along — Pipeline and Routing patterns | 65 min |
| 4 | Langflow hands-on — Reflection and Tool Use patterns | 40 min |

---

## 1. Eight Agent Design Patterns Cheat Sheet

| # | Pattern | Definition | When to Use | Real-World Analogy | Best Demo Tool |
|---|---------|-----------|-------------|-------------------|----------------|
| 1 | **Pipeline / Prompt Chaining** | Sequential chain where each step's output feeds the next step's input. | When a task breaks into ordered stages (e.g., extract, score, act). | Assembly line — each station adds value before passing the part forward. | Make.com |
| 2 | **Routing** | A decision node directs input to one of several downstream paths based on conditions. | When different inputs require different handling (e.g., intent-based triage). | Airport immigration — separate lanes for citizens, residents, and visitors. | Make.com |
| 3 | **Parallelization** | Multiple sub-tasks execute simultaneously, then results are aggregated. | When independent analyses can run at the same time to save latency. | A team of reviewers each grading a different section of a proposal at the same time. | Make.com / Langflow |
| 4 | **Reflection** | The agent critiques its own output and iterates to improve it before finalizing. | When output quality matters and you can afford an extra LLM call (writing, code, analysis). | An author writing a draft, then re-reading it as an editor before submitting. | Langflow |
| 5 | **Tool Use** | The agent calls external tools (APIs, databases, calculators) to get information it cannot generate from memory. | When the task requires real-time data, computation, or actions in external systems. | A chef who checks a thermometer rather than guessing if the oven is hot enough. | Langflow |
| 6 | **MCP (Model Context Protocol)** | A standardized protocol that lets agents discover and invoke tools dynamically via a universal interface. | When you want plug-and-play tool integration without hard-coding each API. | USB-C — one standard port that works with any compatible device. | Conceptual (demo in later sessions) |
| 7 | **Planning** | The agent decomposes a complex goal into a sequence of sub-goals before executing. | When the task is open-ended and requires a multi-step strategy (e.g., research projects). | A project manager creating a work breakdown structure before assigning tasks. | Langflow / Claude |
| 8 | **Multi-Agent Collaboration** | Multiple specialized agents work together, each handling a distinct role, coordinated by an orchestrator. | When no single prompt/agent can handle the full scope and specialization improves quality. | A cross-functional product squad — PM, designer, and engineer each contribute expertise. | Conceptual (demo in later sessions) |

---

## 2. Key Concepts Quick Reference

### Agentic AI vs Generative AI vs Predictive AI

| Dimension | Predictive AI | Generative AI | Agentic AI |
|-----------|--------------|---------------|------------|
| **Core task** | Classify or forecast from historical data | Generate new content (text, image, code) | Autonomously pursue goals via reasoning + action |
| **Interaction** | Input in, prediction out (one-shot) | Prompt in, content out (one-shot or conversational) | Goal in, multi-step execution with tool calls |
| **Autonomy** | None — deterministic output | Low — follows the prompt | High — decides what to do next |
| **Memory** | Training data only | Context window | Context window + external memory / state |
| **Examples** | Churn prediction, fraud detection | ChatGPT drafting an email | An agent that researches competitors, drafts a report, and emails it to stakeholders |

### What Makes an Agent "Agentic"

An AI system is agentic when it exhibits these four capabilities:

- **Autonomy** — Can decide what step to take next without human instruction at each step.
- **Tool Use** — Can call external systems (APIs, databases, browsers) to gather info or take action.
- **Memory** — Can retain and reference information across steps or sessions.
- **Planning** — Can decompose a goal into sub-tasks and sequence them.

### Single-Agent vs Multi-Agent Systems

| Aspect | Single Agent | Multi-Agent |
|--------|-------------|-------------|
| Architecture | One LLM loop handles everything | Multiple specialized agents coordinated by an orchestrator |
| Complexity | Lower — easier to build and debug | Higher — requires coordination logic |
| Best for | Well-scoped tasks with clear steps | Complex workflows needing diverse expertise |
| Failure mode | Single point of failure | Can isolate failures to one agent |
| PM guidance | Start here (MVA) | Graduate to this when single-agent hits limits |

---

## 3. Make.com Exercise Instructions

### Exercise 1: Pipeline Pattern — Intent Scoring Pipeline

**Goal:** Build a webhook-triggered pipeline that receives a user message, scores the intent using AI, and passes the result to a router.

**Steps:**

1. **Create a new Scenario** in Make.com.
2. **Add Module 1 — Webhooks > Custom Webhook**
   - Click "Add" to create a new webhook and copy the URL.
   - Send a test JSON payload: `{ "name": "Alex", "message": "I need pricing for the enterprise plan ASAP" }`
   - Click "Run once" and send the test payload to define the data structure.
3. **Add Module 2 — OpenAI > Create a Completion (or Chat)**
   - Connect your OpenAI API key.
   - System prompt:
     ```
     You are an intent scoring agent. Given a user message, respond with ONLY a JSON object:
     { "intent_score": "high" | "medium" | "low" | "risky", "reason": "<brief explanation>" }
     ```
   - User message: Map the `message` field from the webhook.
4. **Add Module 3 — JSON > Parse JSON**
   - Input: Map the AI module's output text.
   - This converts the string response into usable fields.
5. **Add Module 4 — Router**
   - Connect the Router module after Parse JSON.
   - You will configure the Router paths in Exercise 2.
6. **Test** the pipeline end-to-end by sending a request to your webhook URL.

### Exercise 2: Routing Pattern — Intent-Based Triage

**Goal:** Configure the Router to direct messages to different actions based on the scored intent.

**Steps:**

1. **Router Path 1 — High Intent** (Filter: `intent_score` equals `high`)
   - Add Module: **Email > Send an Email** (or Gmail > Send an Email)
   - Configure: Draft a personalized reply referencing the user's name and message.
   - Subject: "Re: Your inquiry" / Body: Use AI output + mapped fields.

2. **Router Path 2 — Medium Intent** (Filter: `intent_score` equals `medium`)
   - Add Module: **HTTP > Make a Request** (simulating adding to a drip campaign)
   - Or use **Google Sheets > Add a Row** to log to a "Drip Queue" spreadsheet.
   - Log: name, message, reason, timestamp.

3. **Router Path 3 — Low Intent** (Filter: `intent_score` equals `low`)
   - Add Module: **Google Sheets > Add a Row** (or Data Store > Add/Replace a Record)
   - Log the message for analytics — no immediate action.

4. **Router Path 4 — Risky** (Filter: `intent_score` equals `risky`)
   - Add Module: **Slack > Create a Message**
   - Channel: Select a test channel.
   - Message: "ALERT: Risky message from {{name}}: {{message}} — Reason: {{reason}}"

5. **Test** each path by sending messages with different intents to the webhook.

**Test payloads to try:**
- High: `"I want to buy the enterprise plan today"`
- Medium: `"Can you send me more information about features?"`
- Low: `"Just browsing, no questions"`
- Risky: `"I'm going to cancel and tell everyone your product is broken"`

---

## 4. Langflow Exercise Instructions

### Exercise 3: Reflection Pattern — Self-Critiquing Writer

**Goal:** Observe and modify a flow where an LLM generates content, a second prompt critiques it, and the original output is revised.

**Steps:**

1. Open the provided Reflection flow template in Langflow.
2. Examine the flow structure:
   - **Prompt 1 (Writer):** Generates a first draft (e.g., a product description).
   - **Prompt 2 (Reviewer/Critic):** Evaluates the draft against criteria (clarity, tone, completeness).
   - **Prompt 3 (Reviser):** Takes the original draft + critique and produces an improved version.
3. **Your task — Modify the Reviewer prompt:**
   - Open the Reviewer prompt node.
   - Add a new evaluation criterion relevant to your product context (e.g., "Does it address the target persona's pain point?").
   - Save and re-run the flow.
4. **Compare outputs:** Look at the original draft vs. the revised draft. Note what the reflection step improved.

**Discussion questions:**
- How does the quality change with a more specific reviewer prompt?
- What is the latency/cost trade-off of adding a reflection step?
- When would you skip reflection in a production agent?

### Exercise 4: Tool Use Pattern — Data-Augmented Agent

**Goal:** Explore an agent that calls an external tool (API) to retrieve live data before generating its response.

**Steps:**

1. Open the provided Tool Use flow template in Langflow.
2. Examine the flow structure:
   - **Agent node** with an attached tool component.
   - **Tool component** configured to call an external API (e.g., a weather API or mock data endpoint).
3. **Run the flow** with a query that requires external data (e.g., "What is the weather in Bangalore?").
4. **Observe the agent trace:**
   - When does the agent decide to call the tool vs. answer from memory?
   - What does the tool call request/response look like?
5. **Experiment:** Try a question the tool cannot answer. How does the agent handle it?

**Discussion questions:**
- How does the agent decide whether to use the tool?
- What happens if the tool returns an error?
- How would you add guardrails to prevent misuse of a tool?

---

## 5. Tool Links

| Tool | URL | Notes |
|------|-----|-------|
| Make.com | https://www.make.com/ | Sign up for a free account; free tier includes 1,000 ops/month |
| Langflow Cloud | https://www.langflow.org/ | Managed cloud version; also available as open-source |
| Langflow GitHub | https://github.com/langflow-ai/langflow | Self-hosted option |
| Google Antigravity | https://antigravity.withgoogle.com/ | PM workflows demo tool |
| OpenAI API Keys | https://platform.openai.com/api-keys | Needed for Make.com AI modules |
| Make.com Documentation | https://www.make.com/en/help | Scenario building reference |
| Langflow Documentation | https://docs.langflow.org/ | Flow building reference |
| Anthropic Claude API | https://console.anthropic.com/ | Alternative LLM provider |

---

## 6. Glossary

| Term | Definition |
|------|-----------|
| **Agentic AI** | AI systems that autonomously pursue goals through iterative reasoning, tool use, and decision-making. |
| **Agent Opportunity Canvas** | A framework for evaluating where agentic AI can add value in a product — mapping tasks, autonomy levels, and guardrails. |
| **Drift** | Gradual degradation in agent performance over time due to changes in data, user behavior, or model updates. |
| **Guardrails** | Constraints and safety checks placed on an agent to prevent undesirable actions (e.g., content filters, action limits, human approval gates). |
| **HITL (Human-in-the-Loop)** | A design pattern where a human reviews or approves agent actions at critical decision points before they are executed. |
| **MCP (Model Context Protocol)** | An open standard that allows AI agents to discover and use external tools and data sources through a universal interface. |
| **MVA (Minimum Viable Agent)** | The simplest agentic workflow that delivers value — analogous to MVP but for agent-driven automation. Start with one pattern, prove value, then expand. |
| **Orchestrator** | A coordinating agent or control layer that delegates tasks to specialized sub-agents and aggregates their outputs. |
| **Pipeline / Prompt Chaining** | A sequential pattern where the output of one LLM call becomes the input of the next, forming a processing chain. |
| **Reflection** | A pattern where the agent evaluates its own output against criteria and iterates to improve quality before returning results. |
| **Routing** | A pattern where a decision node directs input to different processing paths based on classification or conditions. |
| **Tool Use** | The ability of an agent to invoke external APIs, databases, or services to obtain information or take actions beyond text generation. |
| **Token** | The basic unit of text processed by an LLM; roughly 0.75 words in English. Costs and context limits are measured in tokens. |
| **Webhook** | An HTTP callback endpoint that triggers a workflow when it receives a request — used as the entry point for Make.com scenarios. |

---

*Session 1 of 6 — Agent-driven Automation in Products — Institute of Product Leadership*
