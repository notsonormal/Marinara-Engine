# Explanation: Engine Modes

Marinara Engine isn't just a text box; it is a multi-modal engine that changes its entire presentation layer depending on how you want to interact. At its core, it supports three main paradigms: **Conversation**, **Roleplay**, and **Game Mode**. 

While all three modes hit the same underlying API (`/api/chat/:id/turn`), they activate completely different frontend React Components and different default Agent sets.

---

## 1. Conversation Mode (The "Messenger")

**Concept:** Discord-style, text-message DMs.
**Best for:** Casual chats, modern-day settings, long-term simulated friendships.

**How it works structurally:**
- The React frontend strips away all immersive elements like backgrounds, weather, and sprites. It swaps to a UI resembling standard texting apps (bubbles, timestamps, avatars next to messages).
- **Core Agents:** This mode leans heavily on the **Schedule Planner** and **Autonomous Messenger** agents. Characters operate on a simulated 24/7 clock, knowing when they should be "asleep" or "at work".
- **Unique Features:** Instead of describing what they are looking at in asterisks, characters can use the **Illustrator** agent to generate "Selfies" and send them to you inline in the chat history, mimicking a real digital conversation.

---

## 2. Roleplay Mode (The "Visual Novel")

**Concept:** Immersive, narrative-driven traditional roleplay.
**Best for:** Detailed story weaving, fan-fiction, rich sensory experiences.

**How it works structurally:**
- The frontend loads the `Sprite Engine` and `Scene Graph`. The chat log is pushed to an overlay, and the dominant visual is the current background scene and the Character's expression sprite standing in front of you.
- **Core Agents:** The **Expression Engine** and **Background** agents are critical here. Every time the LLM returns text, the engine analyzes the sentiment (Angry, Blushing, Sad) and forces the UI to hot-swap the sprite on the screen.
- **Unique Features:** Characters don't usually send "Selfies"; instead, the environment itself shifts. Snow overlays, rain particles, and dynamically changing times of day create an immersive "in-world" feel.

---

## 3. Game Mode (The "Tabletop RPG")

**Concept:** A system-driven, multi-character statistical roleplay.
**Best for:** D&D-style campaigns, point-and-click logic, tracking stats.

**How it works structurally:**
- Game mode combines the visual immersiveness of Roleplay mode but adds a distinct "System" layer. Instead of purely chatting with one entity, an **AI Game Master** manages a "Party" of characters.
- **Core Agents:** Relies heavily on the **Combat**, **Quest Tracker**, and **World State** agents. Combat agents parse attacks and automatically roll virtual dice to update HP pools. 
- **Unique Features:** 
  - **Party Card Sidebar:** The UI exposes a sidebar tracking your party members' levels, abilities, and items.
  - **Dialogue Tracking:** Instead of just generating one block of text, the AI Game Master correctly formats dialogue so the UI knows exactly which NPC is speaking, throwing their specific portrait next to their lines. 
  - **CYOA (Choose Your Own Adventure):** When enabled, after the GM responds, an agent generates 2-4 structured "choices" the player can click on to respond, rather than typing manually.
