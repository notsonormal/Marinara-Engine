# Explanation: Understanding Presets

In Marinara Engine, "Presets" are reusable bundles of configurations. Because the engine supports multiple game modes, different backend providers, and highly customizable prompt structures, configurations are split into distinct layers. 

The two primary types of presets you will interact with are **Chat Presets** and **Prompt Presets**. Understanding the difference between them is crucial for managing your roleplay environments.

## 1. Chat Presets

Chat Presets are high-level bundles of settings that define the **starting state** for a newly created chat. There is one "active" Chat Preset per Chat Mode (Conversation, Roleplay, Game) that dictates how new chats in that mode behave by default.

### What Chat Presets DO Carry
When you save or apply a Chat Preset, it bundles the mechanical configuration of the chat:
- The selected **Connection** (which AI provider/backend to use).
- The selected **Prompt Preset** (how the prompt should be formatted).
- Advanced parameters (context limits, memory recall settings).
- Selected Agents and Tools (e.g., Narrative Director, Web Search).
- Specific configurations like translation toggles or Discord mirror settings.

### What Chat Presets DO NOT Carry
Chat Presets do not save the "identity" or narrative content of a chat. They intentionally exclude:
- The Chat Name and Summary.
- Selected Characters or Personas.
- Group configurations and Scene lifecycle states.
- Bound Lorebooks (Lorebooks are owned by the chat, not the preset).
- Chat History.

**Summary:** Think of a Chat Preset as your "Hardware Configuration." It tells the engine *how* to run the chat, but not *who* is in it or *what* it is about.

## 2. Prompt Presets

While Chat Presets handle the engine-level settings, **Prompt Presets** handle the exact text structure sent to the AI Model. They are templates that dictate how character definitions, world info, and chat history are assembled into a final prompt.

### Key Components of a Prompt Preset
- **Section Order:** Defines the exact order in which blocks (like Main System Prompt, Character Description, Scenario, Chat History) are injected.
- **Generation Parameters:** Defines the AI model's generation settings (Temperature, Top P, Max Tokens, Frequency Penalty, Reasoning Effort).
- **Wrap Formats:** Dictates whether the prompt sections are wrapped in XML tags (`<system>`, `<char>`), Markdown headers (`## Description`), or left bare.
- **Variables / Choice Blocks:** Allows for dynamic toggles (like POV selectors or style options). Users can pick an option per chat, which replaces variables like `{{POV}}` in the prompt template.

**Summary:** Think of a Prompt Preset as your "Prompt Engineering Template." It controls the *format* and *style* of the text sent to the LLM.

## 3. The "Chat History" Marker and Role Shifting

In the Prompt Preset UI, you will notice a block labeled **Chat History** with a `MARKER` badge. This block acts as a dynamic structural anchor. 

Although the UI may list it under "System" blocks, the engine's assembler (`assembler.ts`) uses this marker to split your prompt and apply strict role formatting (`system`, `user`, `assistant`).

### How the Final Payload is Structured

When the engine encounters the Chat History marker, it injects the actual array of past back-and-forth messages. The placement of this marker dictates how the surrounding instruction blocks are handled:

1. **Blocks ABOVE the Marker (The "System" Block):** 
   Any instruction blocks (like Role, Setting, Characters) placed above the chat history are concatenated into a single, massive `system` message at the top of the prompt.
2. **The Marker Itself:** 
   Is replaced by the multi-turn history (retaining original `user` and `assistant` roles).
3. **Blocks BELOW the Marker (The Post-History Shift):** 
   Any instruction blocks (like Output Format or World State Directives) placed *below* the chat history are dynamically shifted from `system` to `user`. The engine then merges these instructions into the final `user` message of the chat history.

**Why does it shift roles?**
APIs like Anthropic require the final message to be from the `user`. Furthermore, LLMs suffer from "lost in the middle" syndrome. By allowing you to place "Output Format" *below* the chat history, the engine sneaks your most critical formatting rules into the final user turn. This ensures the AI reads them right before generating, guaranteeing it follows the format.

## How They Work Together

1. You create a new Chat in **Roleplay Mode**.
2. The engine looks at the **active Chat Preset** for Roleplay Mode.
3. The Chat Preset says, "Use the *Claude 3.5 Sonnet* Connection and the *SillyTavern XML* **Prompt Preset**."
4. You select a Character and start chatting.
5. When you send a message, the engine uses the **Prompt Preset** to format the Character's description, the chat history, and the system instructions into a single ChatML structure.
6. The engine sends this formatted prompt through the **Connection** specified by the Chat Preset.

This separation of concerns allows you to swap Characters without changing the AI model, or swap AI models without breaking your carefully tuned prompt format!
