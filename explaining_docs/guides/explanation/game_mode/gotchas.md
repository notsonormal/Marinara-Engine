# Deep Dive: Known Gotchas in Game Mode

This document acts as the mortuary. If a user brings up a weird bug in Game Mode, check these fragile zones first.

### 1. Combat Desyncs
**The Issue:** The user clicked "Attack", the UI says "Hit!", but the NPC's health bar didn't drop.
**The Cause:** The `combat.service.ts` successfully rolled the dice, but the API returned without committing to `checkpoints.ts` (perhaps due to a DB lock). 
**Fix Direction:** Check the Drizzle ORM transaction block in the agent runner to make sure the HP update wasn't caught in a silent SQL rollback loop.

### 2. Dialogue Speaker Misattribution
**The Issue:** The LLM's dialogue text is showing up attached to the wrong Character Avatar in the chat UI.
**The Cause:** Game Mode relies on the LLM formatting its output perfectly (e.g., `Speaker Name: "Dialogue"`). If you are using a smaller, less capable LLM (like Llama 3 8B), it may fail to write the speaker name properly, causing the `GameDialogueOverlay.tsx` regex matching to fail and assign the dialogue to an "Unknown" blank avatar.
**Fix Direction:** Force the user onto a smarter LLM, or write a more robust fallback regex parser in the Client component.

### 3. The "Infinite Loading" Spinner
**The Issue:** The UI is stuck spinning and no text appears.
**The Cause:** This is highly pronounced in Game Mode because so many pre-generation Agents execute. If exactly *one* agent throws an unhandled Promise rejection (for instance, the `Spotify DJ` agent fails to reach the Spotify API due to network timeout), the entire runner crashes before the LLM is ever called.
**Fix Direction:** Ensure every single agent inside `agent-executor.ts` is wrapped in an iron-clad `try/catch` block that defaults to `success: false` but allows the prompt loop to continue running.
