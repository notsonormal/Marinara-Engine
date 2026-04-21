# Deep Dive: Execution Trace of a Roleplay Turn

Roleplay Mode relies on heavy Client-side cinematic elements. When a user sends a message, they aren't just looking for text back; they expect the environment (weather, sprites, music) to shift dynamically based on what the story requires.

## The Execution Path

### 1. The Frontend Trigger
* **Location:** `packages/client/src/components/chat/ChatInput.tsx`
* **Action:** The user dictates a detailed narrative action.
* **Component Layer:** The overarching container is `ChatRoleplaySurface.tsx` combined with `RoleplayHUD.tsx`.
* **State Dispatch:** The prompt is sent via `POST /api/chat/:id/turn` through `React Query`.

### 2. The Server API Entry
* **Location:** `packages/server/src/routes/chat.turn.ts`
* **Action:** Route receives the message body. Dispatches to the `Agent Runner`.

### 3. Pre-Generation (World State Interception)
* **Location:** `packages/server/src/services/agents/`
* **Action:** 
  - **`World State` Agent:** Reads the active location, time of day, and weather. It constructs a scene-setting prompt payload to enforce continuity (e.g. `[SCENE: Forest Clearing. Weather: Raining. Time: Twilight.]`).
  - **`Lorebook Engine`:** Very active in Roleplay mode. Scans the user's dense writing for keywords to retrieve massive contextual world-building texts before the generation begins.

### 4. Prompt Assembly 
* **Location:** `packages/server/src/services/prompt/pipeline.ts`
* **Action:** The prompt is compiled. Unlike Game Mode, there are no dice roll mechanics injected. Unlike Chat Mode, the prompt explicitly encourages the LLM to write in long-form prose and describe actions inside asterisks `*like this*`. 

### 5. LLM Call
* **Location:** `packages/server/src/services/llm/`
* **Action:** A standard completion call streaming from Anthropic/OpenAI/etc.

### 6. Post-Generation (The Visual Engine)
* **Location:** `packages/server/src/services/agents/`
* **Action:** This is where Roleplay mode shines.
  - **`Expression Engine` Agent:** Analyzes the output block (e.g. `*She gasped in shock, dropping the glass*`). It classifies the sentiment as `Surprise/Shock`, and embeds an instruction to load the `shocked.png` sprite over the default idle sprite.
  - **`Background / Weather` Agent:** If the LLM just narrated walking indoors from out of the rain, this agent detects the transition, disables the Rain overlay flag, and triggers a background image swap.
  - **`Spotify DJ` Agent:** Checks if the mood dramatically shifted. If a fight broke out, it reaches out to the user's Spotify Desktop connection to change the playlist to `Battle Music`.

### 7. Client Rehydration (Cinematic Execution)
* **Location:** `packages/client/src/components/chat/ChatRoleplaySurface.tsx`
* **Action:** 
  - The payload hits the Frontend. The new text is displayed in `ChatMessage.tsx`.
  - The UI triggers side-effects via Zustand `ui.store.ts`:
    - `SpriteOverlay.tsx` actively swaps out the character PNG image to the `shocked.png` expression.
    - `WeatherEffects.tsx` fades out the rain particles via `Framer Motion`.
    - `SceneBanner.tsx` displays an elegant pop-up if the location shifted!
