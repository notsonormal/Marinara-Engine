# Built-In AI Agents Reference

Marinara Engine uses an **Agent Runner** to execute modular prompts before, during, and after a chat turn. This allows the system to autonomously control sub-systems (like RPG stats, visuals, and music) without bloating the main character prompt.

This document provides both a high-level list of all agents and a technical deep-dive into their internal logic.

## 📂 Related Files
*   **Prompts**: [agent-prompts.ts](../../packages/shared/src/constants/agent-prompts.ts) (Default templates for all agents)
*   **Configs**: [agent_configs.json](../../packages/server/data/storage/tables/agent_configs.json) (Database entries for built-in agents)
*   **Execution**: [agent-executor.ts](../../packages/server/src/services/agents/agent-executor.ts) (Logic for running agent prompts)
*   **Pipeline**: [agent-pipeline.ts](../../packages/server/src/services/agents/agent-pipeline.ts) (Orchestration of pre/post turn agents)
*   **Local Sidecar**: [sidecar-inference.service.ts](../../packages/server/src/services/sidecar/sidecar-inference.service.ts) (Internal logic for the local agent model)


---


## 🏗️ Core Tracker Agents
These agents analyze the narrative to maintain the simulation's state. They usually run in the **Post-Generation** phase to update data for the next turn.

| Agent | Purpose | Phase |
|-------|---------|-------|
| **World State** | Tracks the current date, time, weather, and physical location. | Post |
| **Character Tracker** | Tracks which characters are present, their current mood, outfit, and internal thoughts. | Post |
| **Persona Stats** | Monitors the player's needs (Satiety, Energy, Hygiene) and inventory. | Post |
| **Custom Tracker** | Tracks user-defined fields like currency, reputation, or custom counters. | Post |
| **Quest Tracker** | Identifies new objectives, completion states, and rewards from the story. | Post |

### Technical Deep Dive: Trackers
*   **World State**: Scans the assistant's message for hints about time, weather, and location. It is instructed to "infer" defaults if the text is vague (e.g., a tavern at night → "Cool", "Clear skies"). Output: JSON containing `date`, `time`, `location`, `weather`, and `temperature`.
*   **Character Tracker**: Identifies which characters are in the scene. It tracks their current mood, outfit, and even "internal thoughts" they didn't voice. Output: An array of `presentCharacters` with their individual states.
*   **Quest Tracker**: Monitors for "Quest Given" or "Objective Met" signals. It maintains a list of up to 3 active quests with checklists and rewards.
*   **Combat Tracker**: Tracks HP, initiative order, and combat status (active/fled/dead) during encounters. It estimates damage based on narrative severity.
*   **Persona Stats**: Tracks the player persona's physical and mental well-being (not combat stats). It adjusts values proportionally to events (e.g., eating a snack → +5-10% satiety).


---

## ✍️ Narrative & Writing Agents
These agents steer the "quality" and "direction" of the prose. They typically run **Pre-Generation** to inject instructions into the final prompt.

| Agent | Purpose | Phase |
|-------|---------|-------|
| **Prose Guardian** | Analyzes recent messages to provide writing directives (e.g., "Avoid smirking", "Use more sensory details"). | Pre |
| **Narrative Director** | Injects stage directions to fix pacing issues or force drama when the story gets stale. | Pre |
| **Secret Plot Driver** | A hidden architect that manages long-term story arcs and subtle plot twists. | Pre |
| **CYOA Choices** | Generates 2-4 plausible in-character choices for the player to make next. | Post |
| **Director** | Specifically focuses on pacing and ensuring the user isn't sidelined in the narrative. | Pre |

### Technical Deep Dive: Narrative
*   **Prose Guardian**: Studies the last 5 messages for overused words or repetitive sentence structures. It produces a "Banned List" and picks 1-2 "Rhetorical Devices" (like Metaphor or Alliteration) to use next. Output: Directives for `BANNED ELEMENTS`, `SENTENCE STRUCTURE`, and `SENSORY FOCUS`.
*   **Narrative Director**: Injects "Director's Notes" (e.g., `[Director's note: The tavern door should burst open]`) to fix pacing or introduce new stimuli when the scene gets stale.
*   **Secret Plot Driver**: Manages two layers: a long-term **Overarching Arc** and a short-term **Scene Direction**. It detects "staleness" (when the conversation goes in circles) and triggers events to break the loop. Output: JSON with `overarchingArc`, `sceneDirections`, and `pacing` mode.
*   **CYOA Choices**: Generates 2–4 short, in-character choices written in the first person. Each choice must be meaningfully different and plausibly driven by the persona's personality.


---

## 🎨 Visual & Media Agents
These agents bridge the gap between text and multimedia.

| Agent | Purpose | Phase |
|-------|---------|-------|
| **Expression Engine** | Detects character emotions and switches their sprite expressions (Happy, Angry, etc.). | Post |
| **Background** | Automatically switches the UI background image to match the current location. | Post |
| **Illustrator** | Identifies visually significant moments and generates prompts for AI image services. | Post |
| **Spotify DJ** | Analyzes the scene's mood and controls Spotify playback/playlists. | Post |
| **HTML Injector** | Injects custom CSS/HTML (like terminals or letters) into the chat when appropriate. | Utility |

### Technical Deep Dive: Visuals & Media
*   **Expression Engine**: Analyzes emotional subtext and compares it against character `<available_sprites>`. Output: JSON mapping `characterId` to an `expression` and a `transition` (crossfade, bounce, shake).
*   **Spotify DJ**: Scans for mood/genre cues and uses tools (`spotify_search`, `spotify_play`) to control music. It prioritizes the user's "Liked Songs" library for better immersion.
*   **Illustrator**: Generates high-quality image prompts only when a scene is visually significant (new location, major reveal). Includes full physical descriptions of all present characters.
*   **HTML Injector**: Detects opportunities to show "in-world" documents (letters, posters, hacking terminals) and injects custom CSS/HTML directly into the chat flow.


---

## 🛡️ Maintenance & Integrity Agents
These agents handle the "janitorial" work of keeping the roleplay consistent.

| Agent | Purpose | Phase |
|-------|---------|-------|
| **Continuity Checker** | Flags factual contradictions (e.g., a character being in two places at once). | Post |
| **Editor** | A final-pass agent that polishes the generated response to fix names, stats, or repetitions. | Post |
| **Prompt Reviewer** | Scores the final assembled prompt for coherence before it's sent to the LLM. | Pre |
| **Lorebook Keeper** | Dynamically creates or updates Lorebook entries as new facts are established. | Post |
| **Card Auditor** | Detects when a character's personality or description has "evolved" and suggests card edits. | Post |

### Technical Deep Dive: Integrity
*   **Continuity Checker**: Compares the latest message against established lore and history. Looks for travel errors, name mix-ups, or item contradictions.
*   **Editor**: A "surgical" agent. It receives the draft response and all agent data, then edits only the roleplay text to fix errors while preserving the original voice and tone.
*   **Lorebook Keeper**: Analyzes narrative for durable new facts. It dedupes against existing entries and only appends `newFacts` to avoid erasing user-written content.
*   **Prompt Reviewer**: Analyzes the final assembled system prompt *before* generation. It catches broken XML tags, conflicting rules, and redundant instructions to save tokens.


---

## 🛠️ Utility & Knowledge Agents
Specialized agents for background processing and world-building.

| Agent | Purpose | Phase |
|-------|---------|-------|
| **Echo Chamber** | Simulates a live-streaming chat with viewers reacting to the story beats. | Post |
| **Chat Summary** | Generates rolling summaries of the conversation to save context space. | Post |
| **Knowledge Router** | Decides which Lorebook entries are relevant to the current turn. | Pre |
| **Knowledge Retrieval** | Extracts and summarizes facts from the selected Lorebook entries. | Pre |
| **Haptic Control** | Translates intimate descriptions into commands for connected hardware (Buttplug.io). | Post |
| **Schedule Planner** | Manages character routines and locations based on the in-game time. | Pre |

### Technical Deep Dive: Utility
*   **Knowledge Router & Retrieval**: The Router picks IDs from the catalog; the Retrieval agent then condenses those entries into the minimum text needed to inform the next turn.
*   **Echo Chamber**: Generates viewer personas (e.g., `xX_Shadow_Xx`) and short, chaotic reactions (hype, memes, criticism) displayed in a scrolling UI sidecar.
*   **Haptic Control**: Translates physical/intimate descriptions into vibration and pattern commands for Buttplug.io compatible hardware.
*   **Chat Summary**: Generates rolling, incremental summaries of the plot to prevent the chat's context window from overflowing.


---

## ⚙️ Architectural Concepts

### The "Sidecar" & Local Inference
To keep costs low and performance high, Marinara Engine is designed to run its agents on a **Sidecar Model** (typically a small local model like **Gemma-2b** or **Qwen-1.5b**).
*   **Cost Efficiency**: 90% of agents run on the local sidecar, meaning they don't cost any API credits.
*   **Parallelization**: Agents run in the background while the main LLM is still "thinking" or streaming.
*   **Privacy**: State tracking and narrative analysis happen entirely on your local machine.

### Execution Phases
The timing of an agent determines how it can affect the story:
1.  **Pre-Generation (Steering)**: Runs *before* the main LLM starts writing. These agents (like **Prose Guardian** or **Knowledge Router**) inject hidden instructions or lore into the prompt to "steer" the AI's response.
2.  **Parallel (Observation)**: Runs at the same time as the main LLM. These agents (like **Illustrator**) don't need to see the AI's response to do their work.
3.  **Post-Generation (State & UI)**: Runs *after* the AI finishes writing. These agents (like **Character Tracker** or **Expression Engine**) analyze the completed message to update the world state, change character outfits, or switch background music.

---

## 🧠 Data Flow: Character Library & Consistency

### The "Broad" vs "Narrow" View
One of the engine's most important optimizations is how it handles character cards to prevent "character bleed" and save tokens:

*   **Main LLM (Narrow View)**: Only sees the full character cards for characters you have specifically added to the chat. If a character isn't in the scene, the main AI doesn't know they exist.
*   **Agents (Broad View)**: The Sidecar agents have a "Master List" of your entire character world (Name + 2000-character description). This allows them to act as a **Reference Library**.

### Introduction Safety Nets
What happens if the AI suddenly introduces a character that isn't active in the chat yet?
1.  **Phase 1: Knowledge Router (Pre-Gen)**: If a character is mentioned by name in your prompt, the Router "hooks" their Lorebook entry into the prompt *before* the response starts.
2.  **Phase 2: Character Tracker (Post-Gen)**: If the character appears mid-story, the Tracker identifies them from the "Master List" and officially marks them as "Present."
3.  **Phase 3: Scene Promotion**: For the *next* turn, the engine sees they are "Present" and automatically "promotes" their full Character Card into the Main LLM's prompt.
4.  **Phase 4: Card Evolution Auditor**: If the AI improvised a new detail during the intro (e.g., "Dottore is wearing a red scarf"), the Auditor asks you if that should become a permanent part of the character's card.

---

## 🛠️ Agent Side-Effects & Tool Usage
Agents aren't just for text; they can trigger real-world and UI actions using **Tools**:

*   **UI Updates**: Switching background images, changing sprite expressions, or displaying HTML widgets (like a hacking mini-game terminal).
*   **Metadata Patches**: Agents can "patch" the hidden metadata of a chat. For example, a **Secret Plot Driver** can record that the user found a hidden key without the user ever seeing that data in the chat.
*   **External Hardware**: The **Spotify DJ** controls your music player, and **Haptic Control** sends physical pulses to Buttplug.io connected hardware.

---

## 🚦 Model Recommendation Guide

Choosing the right model for each agent is a balance between **Cost/Speed** and **Reasoning Quality**.

| Agent Category | Specific Agents | Recommended Model | Rationale |
| :--- | :--- | :--- | :--- |
| **State Trackers** | World State, Character Tracker, Persona Stats, Custom Tracker | **Sidecar (2B)** | **Fact Extraction**. These agents just need to pull variables into JSON. Minimal reasoning required. |
| **Active Simulation** | Quest Tracker, Combat Tracker | **Main Light (8B)** | **Game Logic**. These need to "judge" if an objective was met or estimate damage, which requires better context comprehension. |
| **Narrative Steering** | Prose Guardian, Director, Secret Plot Driver | **Main Light (8B)** | **Creativity**. High-quality steering requires a model that understands subtext and pacing, not just raw text. |
| **Integrity & Review** | Continuity, Auditor, Prompt Reviewer | **Main Light (8B)** | **Fact Checking**. Must be smart enough to spot subtle contradictions between the story and the character cards. |
| **The Editor** | Editor | **Main Light+ (8B-70B)** | **Prose Quality**. The only agent that rewrites the story. High-end models are best to preserve the original writer's "soul." |
| **Knowledge Mgmt** | Router, Retrieval, Lorebook Keeper, Chat Summary | **Main Light (8B)** | **Compression**. Summarizing lore without losing the "important" bits requires solid reasoning. |
| **Visuals & Media** | Expression, Background, Spotify DJ, Haptic | **Sidecar (2B)** | **Pattern Matching**. Mapping "mood" to "song" or "sprite" is a simple classification task. |
| **Experimental/UI** | Illustrator, Echo Chamber, HTML, CYOA | **Main Light (8B)** | **Generative Output**. These produce content (prompts, viewer comments, choices) that needs to feel "human" and varied. |

### 💡 The "Rule of Thumb"
*   **JSON-only extraction?** Use your local **Sidecar**.
*   **Rewriting story prose?** Use a **Main Light** (API) or **8B-14B** (Local).
*   **Managing game/quest logic?** Use a **Main Light** to prevent "hallucinated" progress.
*   **Steering the plot?** Start with a **Sidecar**, and upgrade if the directions feel repetitive.



