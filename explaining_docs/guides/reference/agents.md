# Built-In AI Agents Reference

Marinara Engine uses an Agent Runner to execute modular prompts before, during, and after a chat turn. This allows the system to autonomously control sub-systems without bloating the main character prompt.

Below is a reference of the key agents available:

## Pre-Generation Agents
These agents run *before* the main chat LLM is called. Their outputs are injected into the final system prompt.

| Agent | Purpose |
|-------|---------|
| **World State** | Injects the current date, time, weather, and physical location into context. |
| **Quest Tracker** | Reminds the LLM of active user objectives and checks if constraints have been met. |
| **Character Tracker** | Injects dynamic physical states (e.g., current outfit, injuries, mood) into context. |
| **Continuity Checker** | Analyzes the recent chat against the Lorebook to prevent the LLM from writing factual contradictions. |
| **Schedule Planner** | References the time of day and character's habits to determine where they should be located right now. |
| **Response Orchestrator** | Used in Group Chats to autonomously decide *which* specific character should speak next. |

## Post-Generation Agents
These agents run *after* the main chat LLM returns text. They analyze the text to trigger side-effects and UI updates.

| Agent | Purpose |
|-------|---------|
| **Expression Engine** | Analyzes the emotional sentiment of the generated message and forces the UI to switch the character's sprite (e.g., Happy, Angry). |
| **Illustrator** | Looks for strong visual beats in the generated text and automatically fires off a request to the Image Generation provider to create a scene image. |
| **Background** | Evaluates if the location logically changed based on the text and automatically switches the UI background image to match. |
| **Combat** | Looks for attack descriptions. Rolls virtual dice against stats and updates HP trackers. |
| **Love Toys Control** | Scans explicit descriptions and translates intensity into commands sent to hardware via the Buttplug.io service. |
| **Spotify DJ** | Analyzes the mood of the scene and selects an appropriate playlist or track via the Spotify API. |

## System & Maintenance Agents
These agents run asynchronously or are invoked manually to keep the simulation clean.

| Agent | Purpose |
|-------|---------|
| **Chat Summary** | Generates condensed rolling summaries when chats get too long to fit in context. |
| **Prompt Reviewer** | A meta-agent that scores the assembled prompt for coherence before generation. |
| **Lorebook Keeper** | Dynamically creates or updates Lorebook entries if it detects the user established a new, long-term fact during roleplay. |
