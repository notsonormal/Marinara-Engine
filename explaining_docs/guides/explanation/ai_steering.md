# Explanation: AI Steering & Guided Generation

In traditional chat interfaces, the AI follows the "Conversation History." However, in complex roleplay, users often need to manually "steer" the AI's next move without that instruction becoming part of the permanent story text. 

Marinara Engine handles this through a multi-layered steering system designed to feel familiar to users of **SillyTavern's Guided Generation** extension.

## How to Enable

1.  Open the **Settings** panel (cog icon in the sidebar).
2.  Click the **Advanced** tab.
3.  Enable **"Guide generations with chat input"**.

## 1. Guide Generations (Instruction Mode)

When the **"Guide Generations"** setting is enabled for a chat, the main input box changes its behavior **only during regenerations or manual triggers**.

Instead of sending your text as a new message, if you hit **Regenerate** or **Trigger Character Response** while text is in the box, the engine uses that text as a high-priority system instruction for that specific turn.

### Triggers:
*   **Regenerate (Reroll)**: Clicking the reroll icon on an existing assistant message.
*   **Character Picker**: Clicking a character face in the manual trigger menu.

Hitting **Enter** normally will still send your text as a standard user message, allowing you to RP without toggling the setting off.

### Technical Implementation
The engine takes your input and wraps it in a "Consideration" directive. This is pushed as the **final message** in the message array sent to the LLM:

```typescript
// From generate.routes.ts
const instruction = `Take the following into special consideration for your next message: ${input.generationGuide}`;
runningMessages.push({ role: "system", content: instruction });
```

### Why it's at the end
LLMs have a "recency bias." By placing your steering instruction at the absolute bottom of the prompt (after the chat history), the engine ensures the model prioritizes your latest command over the general character definitions or earlier story beats.

## 2. The Narrator Command (`/narrator`)

While Guided Generation is "invisible" steering, the `/narrator` command is **visible steering**.

*   **Behavior**: When you type `/narrator <instruction>`, the engine adds a permanent message to the chat history.
*   **Role**: It uses the `narrator` role (often rendered as a system block or italics).
*   **Technical Implementation**: It wraps your text in a strict instruction block that prevents the AI from assuming the user is speaking:
    > `[Narrator instruction - do not include a reply from {{user}}. Instead, write the next part of the narrative steering it toward the following: {{args}}]`

### When to use which?
| Feature | Visibility | Best For... |
| :--- | :--- | :--- |
| **Guided Generation** | Invisible | Rerolling a message to fix a specific AI error or "nudge" a single response. |
| **Narrator Command** | Permanent | Adding a major plot event or change in setting that should be remembered by the AI forever. |

## 3. The Injections Pipeline

Unlike basic chat apps, Marinara uses **Agents** (Narrative Director, Prose Guardian, etc.) that "inject" text into the prompt silently. 

For users who want total control, the **Injections Tab** (found in the Roleplay HUD -> Agents Menu) exposes these hidden layers.
*   **Manual Overrides**: You can see exactly what the Narrative Director "told" the AI to do and edit it.
*   **Persistent Steering**: If you edit a cached injection and hit "Regenerate," the AI will re-run that specific turn using your modified instructions.

## 3. Impersonation (`/impersonate`)

While "Guided Generation" tells the AI what to do, **Impersonation** tells the AI to *be* the character for a moment.
*   When you use `/impersonate [action]`, the engine constructs a specific prompt designed to force a response in your persona's voice.
*   This is functionally equivalent to SillyTavern's "Impersonate" button but integrated directly into the command line.

## 4. Summary Table for SillyTavern Users

| SillyTavern Feature | Marinara Engine Equivalent | Location |
| :--- | :--- | :--- |
| **Guided Generation** | **Guide Generations** | Input Box Toggle / Settings |
| **Extensions Context** | **Writer Agents (Injections)** | HUD -> Agents -> Injections |
| **Author's Note** | **Durable Author's Note** | HUD -> Info -> Author's Note |
| **Impersonate** | **`/impersonate`** | Chat Input |
