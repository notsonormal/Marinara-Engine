# How to Create a Custom Agent

Marinara Engine's agency loop is entirely data-driven. You don't need to write complex Fastify middleware to create a new Agent. Agents are injected by defining an `agent_config` and linking it to the workflow.

## Step 1: Define the Agent Configuration
The `agentConfigs` table holds the rules for an Agent. Provide a JSON-compatible row with:

- `name`: A human-readable name (e.g., "Sarcasm Detector").
- `type`: A unique identifier string for the runner.
- `phase`: Determines *when* it runs. Valid phases are `pre_generation`, `parallel`, or `post_processing`.
- `promptTemplate`: The system prompt to send to the LLM (e.g., "Analyze the following message and output exactly true or false if it is sarcastic").

## Step 2: Seed the DB (For Core Agents)
If this is an agent you want shipped with the codebase, you should add your new configuration to the seed file located at `packages/server/src/db/seed-mari.ts` (or equivalent data seeder).

## Step 3: Handle the Output (If Necessary)
If your agent simply injects context (`pre_generation`), the `Agent Runner` automatically appends its output to the main context. No code is needed.

However, if your agent causes a **Side Effect** (like changing a UI element or hardware), you must intercept its output in the runner:
1. Open `packages/server/src/services/agents/agent-executor.ts`.
2. Locate the switch statement or map handling `post_processing` agent results.
3. Add a case for your agent's `type` and write the TypeScript logic (e.g., saving a tag to the Database, or invoking a third party API).

## Step 4: Enable in UI
Users can turn agents on and off on a per-chat or global basis. Ensure your new agent appears in the Client's Settings Modal so users can enable it!
