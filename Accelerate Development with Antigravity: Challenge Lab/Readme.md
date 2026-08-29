# Accelerate Development with Antigravity — Challenge Lab

Welcome to the "Accelerate Development with Antigravity" hands-on challenge. This lab walks you through authoring a skill for an Antigravity agent, scaffolding the agent project, and deploying it to Agent Runtime. It's written for learners and students so each step is clear, repeatable, and verifiable.

## Learning objectives

By the end of this lab you will be able to:

- Explain key Antigravity concepts: Agent Runtime, MCP server, and agent skills.
- Author an agent skill that follows a Standard Operating Procedure (SOP).
- Scaffold a new agent project using the agents-cli scaffolding tools.
- Package required local files so the deployment includes them.
- Deploy an agent to Agent Runtime in a specified region.

## Prerequisites

- A Google Cloud project with billing enabled and the required permissions to deploy Agent Runtime applications.
- gcloud SDK installed and authenticated: `gcloud auth login` and `gcloud config set project <PROJECT_ID>`
- agents-cli (or the scaffolding tool used in the lab) installed and on your PATH.
- Local development environment: Python 3.8+ (if the agent uses Python), editor of choice.
- The following file present locally (you will copy it into the agent app folder before deploy): `cymbal_direct_mcp.py`.

If you do not have agents-cli installed, follow your course or project instructions to install it and authenticate to the Antigravity environment.

## Repository layout (this lab)

- Readme.md — this file (lab instructions).
- sample files and prompts — the content you will use to scaffold and configure the agent.

## Background — Key terms

- Google Antigravity: an agentic development platform for building and running autonomous AI agents.
- MCP Server (Model Context Protocol): an adapter layer that connects LLM-based agents to external tools, APIs, and databases.
- Agent Runtime: managed, serverless hosting and execution infrastructure for deploying agents in production.
- Agent Skill: a modular component that defines capabilities, SOPs, and how an agent should respond or call tools.

---

## Lab tasks (step-by-step)

There are three main tasks in this lab: authoring a skill (Task 3), scaffolding the agent (Task 4), and deploying to Agent Runtime (Task 5). Follow each section in order and verify the expected outputs.

### Task 3 — Author an agent skill and workspace rule

Goal: Create a skill named `winter-storm-triage` that defines SOPs for handling customer inquiries about packages delayed by winter storms.

Skill requirements (summary):

1. Order verification
   - Use MCP tools `get_order_status` and `get_customer_loyalty_info` to confirm order state and loyalty tier.

2. Compensation policy
   - Apply compensation according to loyalty tier:
     - Platinum: $100 credit + Next-Day Air upgrade
     - Gold: $50 credit + Next-Day Air upgrade
     - Silver: $25 credit + 3-Day Select upgrade
     - Member: $10 credit + Priority Shipping upgrade

3. Compensation execution
   - Call MCP tool `issue_disruption_compensation` to apply credit and shipping upgrade.

4. Customer communication
   - Generate an empathetic, professional message that:
     - Acknowledges the disruption caused by the winter storm.
     - Explains the resolution (credit and shipping upgrade) clearly.
     - Reinforces the company’s commitment to customer satisfaction.

Suggested prompt (give this to the Antigravity authoring chat or include as the skill description when scaffolding):

"Create an agent skill named `winter-storm-triage` that implements SOPs for winter-storm delay cases. The skill must verify orders using MCP tools `get_order_status` and `get_customer_loyalty_info`, determine compensation per loyalty tier, call `issue_disruption_compensation` to apply refunds/upgrades, and generate an empathetic customer response describing the credit and shipping upgrade. Include configurable parameters for currency amounts, shipping options, and tool names. Provide example success and error responses."

Deliverable for Task 3:

- A skill file (for example `skills/winter_storm_triage.json` or `skills/winter_storm_triage.py`) included in your agent project that implements the above SOPs.

---

### Task 4 — Scaffold the agent project

Goal: Use the agents-cli to generate a new agent project named `winter-storm-triage` and include the skill created in Task 3.

Constraints and instructions:

- Project name: `winter-storm-triage`
- Target local directory: `/home/user/Documents/genai144-challenge/` (change to a path appropriate for your system)
- Model: configure the agent to use `gemini-3.1-flash-lite` in the agent config
- Primary agent configuration file: `app/agent.py` (the scaffolder usually generates this file — inspect it)
- Include a file `implementation_plan.md` in the project root describing how you will implement and test the skill.
- After scaffolding, produce `walkthrough.md` summarizing the scaffolded files, where the skill is located, and any manual changes you made.

Suggested CLI prompt to paste into Antigravity chat to perform scaffolding (or run equivalent commands locally):

Build a new agent project named `winter-storm-triage` using agents-cli skills. Place the project inside `/home/user/Documents/genai144-challenge/`. Configure the agent to use the `gemini-3.1-flash-lite` model. Ensure the project includes the `winter-storm-triage` skill (SOPs from Task 3). Create `implementation_plan.md`. Review the plan and then proceed to authorize the build. Output `walkthrough.md` summarizing the work.

What to check after scaffolding:

- `app/agent.py` exists and references the configured model.
- A `skills/` folder contains `winter-storm-triage` skill files.
- `implementation_plan.md` and `walkthrough.md` are created.

---

### Task 5 — Deploy the agent to Agent Runtime

Goal: Deploy the scaffolded agent to Agent Runtime in region `us-west1` and ensure packaging includes required local files.

Critical file packaging requirement:

- Ensure `cymbal_direct_mcp.py` (the MCP adapter helper) is copied into the agent's `app/` directory before packaging so it will be bundled and available at runtime.

Deployment steps (high-level):

1. In your project root, confirm `app/` contains `agent.py` and `cymbal_direct_mcp.py`.
2. Update deployment configuration to target region `us-west1` (this may be a YAML/JSON config or an option in the Antigravity UI/CLI).
3. Build the deployment bundle using the agents-cli or the packaging tool provided.
4. Deploy to Agent Runtime and monitor the deployment logs for success.

Suggested prompt for the Antigravity UI or CLI:

"Deploy the `winter-storm-triage` agent to Agent Runtime in region `us-west1`. Make sure `cymbal_direct_mcp.py` is copied into `app/` so it is included in the deploy package. Generate an `implementation_plan.md` describing the deployment steps and expected verification checks. Proceed to deploy when ready."

What to verify after deployment:

- The agent service is listed in Agent Runtime (use the console or `gcloud` commands for your environment).
- Deployment status indicates `READY` or `RUNNING` without errors.
- When invoking a test conversation or API call, the agent calls the MCP tool endpoints (or logs show attempts) and returns the expected empathetic response for a simulated delayed delivery case.

---

## Implementation checklist (what to submit/turn in)

- Project folder `winter-storm-triage/` inside your specified local directory.
- `skills/winter_storm_triage` files or equivalent skill artifact implementing the SOPs.
- `app/agent.py` and `app/cymbal_direct_mcp.py` present.
- `implementation_plan.md` describing the plan for authoring, packaging, and deploying (include commands you ran).
- `walkthrough.md` summarizing the scaffold and any manual edits.
- Short demo transcript or screenshots showing a test invocation and the agent’s response (optional but recommended).

## Troubleshooting and tips

- If the deployment fails because a file is missing, check that `cymbal_direct_mcp.py` is inside `app/` before packaging.
- If MCP tool calls fail, verify credentials and network access for the MCP connector and any external APIs.
- Review deployment logs in the Agent Runtime console — they often show missing dependencies and stack traces.

## Grading rubric (suggested)

- Skill correctness (40%): Skill verifies orders, determines loyalty tier, and applies correct compensation.
- Messaging quality (20%): Responses are empathetic, clear, and contain the correct compensation summary.
- Packaging & deployment (30%): Project packaged correctly, `cymbal_direct_mcp.py` present, deployment in `us-west1` succeeds.
- Documentation (10%): `implementation_plan.md` and `walkthrough.md` included and clear.

## Reference prompts (copy/paste-ready)

Task 3 prompt (author skill):

"Create an agent skill named `winter-storm-triage` that implements SOPs for winter-storm delay cases. The skill must verify orders using MCP tools `get_order_status` and `get_customer_loyalty_info`, determine compensation per loyalty tier, call `issue_disruption_compensation` to apply refunds/upgrades, and generate an empathetic customer response describing the credit and shipping upgrade. Include configurable parameters for currency amounts, shipping options, and tool names. Provide example success and error responses."

Task 4 prompt (scaffold):

"Build a new agent project named `winter-storm-triage` using agents-cli skills. Place the project in `/home/user/Documents/genai144-challenge/`. Configure the agent to use model `gemini-3.1-flash-lite`. Ensure the project includes the `winter-storm-triage` skill and create `implementation_plan.md`. Review the plan and proceed to authorize the build. Output `walkthrough.md` summarizing the work."

Task 5 prompt (deploy):

"Deploy the `winter-storm-triage` agent to Agent Runtime in `us-west1`. Ensure `cymbal_direct_mcp.py` is copied into `app/` so it is bundled for deployment. Configure the deployment for the specified region and generate an `implementation_plan.md`. Review and proceed."

---

If you’d like, I can:

- Create the `implementation_plan.md` and `walkthrough.md` templates for you in the project.
- Add a starter `skills/winter_storm_triage` skill file (skeleton) implementing the SOP in JSON or Python.

Tell me which of those you want me to add next and I’ll create the files inside the repository for you.
