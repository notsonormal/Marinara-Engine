# Deep Dive: Execution Trace of a Conversation (Chat) Turn

When a user is in the "Messenger" style view (Conversation Mode) and hits send, the execution path strips out game-like combat rolls and immersive visual checks, focusing instead on time scheduling and autonomous chatting behavior.

## The Execution Path

### 1. The Frontend Trigger
* **Location:** `packages/client/src/components/chat/ConversationInput.tsx`
* **Action:** The user types a text message and hits the send arrow. 
* **State Change:** The input text is trapped. `React Query` fires a mutation (`POST /api/chat/:id/turn`). 
* **Optimistic UI:** The message appears inside `ConversationView.tsx` rendered as a `ConversationMessage.tsx` generic chat bubble.

### 2. The Server API Entry
* **Location:** `packages/server/src/routes/chat.turn.ts`
* **Action:** Identical to all modes. The router picks up the payload and verifies the chat is registered as `conversation` mode, passing it to the core `Agent Runner`.

### 3. Pre-Generation (Time & Schedule Interception)
* **Location:** `packages/server/src/services/agents/agent-executor.ts`
* **Action:** Pre-generation agents run. Because this is a realistic chat simulation:
  - **`Schedule Planner` Agent:** It checks the local time and the NPC's configured sleep/work schedules. If it's 3:00 AM, it might inject a context clue: `[SYSTEM: The user just woke you up in the middle of the night via text.]`
  - **`Autonomous Messenger` Agent:** *(If the turn wasn't triggered by the user, but by a timeout)*, this agent might randomly fire to make the NPC "double text" the user.

### 4. Prompt Assembly 
* **Location:** `packages/server/src/services/prompt/pipeline.ts`
* **Action:** The payload is assembled using a standard conversational prompt. It heavily weights recent chat history over rigid GM instructions to maintain casual banter. Lorebooks are parsed for keywords, pulling in memory files if the user mentions specific topics.

### 5. LLM Call
* **Location:** `packages/server/src/services/llm/`
* **Action:** The string is sent to the LLM Provider. A stream is returned. Note: Regex scripts might run here to strip out asterisks `*` so that the bot's response looks like a standard text message rather than roleplay actions.

### 6. Post-Generation (Selfies & Images)
* **Location:** `packages/server/src/services/agents/agent-executor.ts`
* **Action:** The runner kicks off the post-processing phase.
  - **`Illustrator` Agent:** Analyzes the received text. If the NPC said something like "Check out this dress I just bought!", the agent detects the high visual saliency. It quietly fires off a request to a provider (e.g. `AUTOMATIC1111`) to generate an image. This image URI is appended to the message payload as an attachment. 

### 7. Client Rehydration
* **Location:** `packages/client/src/components/chat/ConversationView.tsx`
* **Action:** 
  - The API responds. `React Query` caches the finalized message row.
  - The UI updates. If the `Illustrator` agent successfully added an image in Step 6, a thumbnail appears inside the bot's chat bubble!
