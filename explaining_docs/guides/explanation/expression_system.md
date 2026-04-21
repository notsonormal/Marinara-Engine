# Explanation: The Expression System

The Expression System in Marinara Engine is responsible for the cinematic, dynamic reaction sprites seen primarily in **Roleplay** and **Game** modes. It bridges the gap between text-based AI generation and visual storytelling.

The system is split into a two-layer architecture: a backend intelligence layer (the Agent), and a frontend presentation layer (the React component).

## 1. Determining the Expression

When the LLM finishes drafting a response (e.g., `*She gasps and drops the glass, her eyes wide.* "What did you say?"`), the engine must classify the sentiment to pick the correct character sprite. It does this via two mechanisms:

### Primary: The Expression Engine Agent
The text payload is handed to the **Expression Engine Agent** on the Fastify server side. 
- This agent acts as an NLP classifier.
- It parses the syntax and determines the over-arching emotion (in this case, "Surprise").
- It dictates *how* the expression should animate (the transition).
- It injects this result back into the message payload: `{ expression: 'surprised', transition: 'hop' }`.

### Secondary: Keyword Parsing Fallback
If the Agent is disabled by the user or fails to execute, the architecture is resilient enough to self-correct on the client side.
- Inside `packages/client/src/components/chat/SpriteOverlay.tsx`, there is a function called `detectExpression(text)`.
- It scans the incoming text for hardcoded regex matches.
- Matches against words like `/b(angry|furious|rage)/i` trigger an "angry" sprite classification entirely locally.

## 2. Executing the Expression Swap

Once the desired expression text is determined, the UI component (`SpriteOverlay.tsx`) must execute the visual transition safely.

### Image URL Resolution
The Client maintains a list of uploaded sprites for the Character. It compares the requested expression (e.g., "Surprised") against the file names.
- If it finds an exact match (like `surprised.png`), it loads it.
- If it cannot find one, it falls back to a substring match.
- If all fails, it silently defaults back to `idle.png` or `neutral.png`.

### Framer Motion Transitions
The sprite swap does not just instantly blink the DOM element out of existence. It uses **Framer Motion** `AnimatePresence` logic to perform cinematic entrances and exits.

The Agent can request four distinct transition animations when passing the new image:
1. **Crossfade (Default):** The new sprite smoothly fades in over 0.4 seconds while the old sprite fades to opacity 0.
2. **Shake:** The sprite violently vibrates on the X-axis (`x: [0, -6, 6, -4, 4, -2, 2, 0]`). Used heavily for "Angry" or "Scared" reactions.
3. **Hop:** The sprite bounces upwards off the bottom of the screen (`y-[0, -18, 0, -8, 0]`). Used mostly for "Surprised" or "Happy".
4. **Bounce:** The sprite scales down to 85% and pops quickly back to 100%, simulating a heartbeat or sudden realization.
