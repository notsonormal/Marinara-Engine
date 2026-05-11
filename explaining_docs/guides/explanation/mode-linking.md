# Explanation: Asymmetric Mode Linking

In Marinara Engine, **Linking** is the mechanism that connects a **Conversation** (Messenger) chat with a **Roleplay** or **Game** chat. This allows characters to exist across different presentation layers while maintaining a shared sense of reality.

Linking is intentionally **asymmetric**. Information flows automatically in one direction (to keep the "Messenger" informed) but requires manual steering in the other (to keep the "Story" clean).

---

## 1. Context Flow: The "Shared World"

### Roleplay → Conversation (Automatic Awareness)
When a Conversation is linked to a Roleplay, the engine automatically injects the Roleplay’s current **summary** and **recent messages** into the Conversation's context.

*   **The Result:** The character in the Messenger chat "knows" what is happening in the story without you needing to explain it.
*   **Use Case:** You are playing a high-fantasy adventure in Roleplay mode. You switch to Conversation mode to text a companion who isn't currently in the scene. Because of the link, they can say, *"I heard you just reached the Silver City! Is the weather as cold as they say?"*

### Conversation → Roleplay (Manual Steering)
Raw messages from the Conversation are **not** injected into the Roleplay. This is a design choice to prevent OOC (Out Of Character) chatter from polluting your story's prose or inflating the token count with meta-talk.

Instead, the link uses a "Steering" model. To bridge information from the Messenger to the Story, characters (or the player) use specific tags:

*   **`<influence>...</influence>`**: A **one-shot** instruction. It is injected into the next Roleplay turn's prompt, consumed by the model, and then removed.
*   **`<note>...</note>`**: A **persistent** instruction. It stays in the Roleplay's context on every turn until you manually clear it from the chat settings drawer.

---

## 2. Cross-Mode Triggers (Commands)

Linking also enables characters to "reach through" the interface and trigger actions in the other mode using hidden commands.

### Initiating a DM from the Story
A character in **Roleplay Mode** can "send a text" to the player using:
`[dm: character="Name", message="Hey, are you still at the tavern?"]`

This trigger will locate the linked Conversation chat and post the message there as the character, mimicking a real-time digital notification from an "in-world" device.

### Initiating a Scene from the DM
A character in **Conversation Mode** can "pull the player into the story" using:
`[scene: scenario="...", background="..."]`

This trigger initiates a narrative scene in the linked Roleplay mode, effectively switching the presentation layer from "texting" to "storytelling" for a specific event.

---

## 3. The `<ooc>` Tag: Breaking the Fourth Wall
In both modes, the `<ooc>...</ooc>` tag acts as a universal "meta-message."

*   In **Roleplay**, characters use it to talk "to the player" about the narrative direction (e.g., *"Wait, did you want me to pick up that sword? <ooc>I'm not sure if I should take it or leave it for later.</ooc>"*).
*   In **Conversation**, it is used for technical talk or direct steering that shouldn't be interpreted as part of the "in-character" texting transcript.

---

## Summary: Why Asymmetric?

By keeping the link asymmetric, Marinara Engine achieves two goals:
1.  **Immersion:** Your story (Roleplay/Game) remains high-quality prose, free from the shorthand and informal style of text messaging.
2.  **Intelligence:** Your characters feel like they inhabit a real world because they don't need you to repeat yourself—they already "know" what happened in the story via the automatic context bridge.
