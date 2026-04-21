# Deep Dive: Execution Trace of a Game Mode Turn

When a user sits at their party UI and clicks "Attack the Goblin", a complex series of events unfolds across the Client UI and the Server API to process combat, update stats, and return a narrative description. If an action in Game Mode fails silently, this trace provides the debugging path to follow.

## The Execution Path

### 1. The Frontend Trigger
* **Location:** `packages/client/src/components/game/GameInput.tsx`
* **Action:** The user selects an action (e.g., "Attack") or types a custom message. 
* **State Change:** The input is captured by the local React component state. A `React Query` mutation is dispatched (`useMutation` in the `chat.store.ts` or `game-mode.store.ts` equivalents) to `POST /api/chat/:id/turn`.
* **Optimistic UI:** The user's bubble appears in `GameSessionHistory.tsx` with a loading spinner.

### 2. The Server API Entry
* **Location:** `packages/server/src/routes/chat.turn.ts` (or equivalent controller)
* **Action:** The Fastify router receives the payload. It queries the Data Layer (`Drizzle ORM`) to load the `chats` row and verify that the chat is currently set to `game` mode.
* **Transition:** Hands the payload over to the `Agent Runner` (`packages/server/src/services/agents/agent-executor.ts`).

### 3. Pre-Generation (Game Logic Interception)
* **Location:** `packages/server/src/services/agents/`
* **Action:** The runner fires the pre-generation phase. 
* **Game Specifics:** Because this is Game Mode, specialized agents are active:
  - **`combat.service.ts`:** If the user selected an attack, this service intercepts the payload, rolls virtual dice against the target's AC (Armor Class), and determines if it's a hit or miss. This raw data (e.g., "Critical Hit, 15 Damage") is stored in the pipeline context.
  - **`time.service.ts` & `weather.service.ts`:** Injects the current hour and environmental conditions into the game context.

### 4. Prompt Assembly for the GM
* **Location:** `packages/server/src/services/game/gm-prompts.ts`
* **Action:** Unlike a standard chat, the prompt pipeline uses the Game Master template.
* **Splicing:** It bundles the Party's stats, the current active NPCs in the scene, and the results from the `combat.service.ts` (e.g. telling the LLM "The player rolled a Critical Hit against the goblin"). It instructs the LLM to narrate the outcome of that hit.

### 5. LLM Call
* **Location:** `packages/server/src/services/llm/`
* **Action:** The context array is sent down the wire to the provider (e.g., OpenAI). The response is streamed back. The LLM acts as the GM, narrating the bloody sword swing over the goblin.

### 6. Post-Generation Updates
* **Location:** `packages/server/src/services/agents/agent-executor.ts` (Handling result)
* **Action:** 
  - The text is saved to `chat_messages`.
  - Side effects calculated in Step 3 are firmly committed to the DB via `checkpoints.service.ts` or `game-state.ts` (e.g., the Goblin's HP array element is permanently lowered).

### 7. Client Rehydration
* **Location:** `packages/client/src/components/game/GameCombatUI.tsx` & `GameSessionHistory.tsx`
* **Action:** 
  - The API returns a `200 OK` with the updated Game State and the new narrative message.
  - `React Query` overwrites the cache.
  - The UI re-renders: the loading spinner vanishes, the GM's narration appears, and `GamePartySidebar.tsx` updates if any player took damage.
