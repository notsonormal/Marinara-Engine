# Deep Dive: Subsystem API Contract - The GM Prompt

In Game Mode, the AI acts as a Game Master governing multiple NPCs rather than roleplaying specifically as a single companion. This requires a very specific construction of the prompt payload inside `packages/server/src/services/game/gm-prompts.ts`.

## The Contract

### Inputs to `gm-prompts.ts`
When the Pipeline triggers the GM prompt assembly, it expects the following objects to exist in the application memory:
1. **The Game State Object:** Current Time, Current Weather, Active Map Node.
2. **The Party Array:** An array containing metadata on the User's characters, their classes, levels, and active debuffs.
3. **Agent Side-Effects:** Any output from `combat.service` (e.g. `[SYSTEM: Player attacked Goblin with Sword. Dice: 18 vs AC 14. HIT for 6 damage.]`)

### Processing (The "Black Box")
The service mathematically concatenates these inputs to force the LLM to understand its boundaries.
- It inserts strict instruction blocks warning the AI not to invent player actions.
- It translates JSON statistical data (like HP and STR) into natural language prose instructions to save tokens.

### Outcomes
The final prompt block returned to the Connection Manager looks like this:

```text
[SYSTEM: You are the Game Master. 
Current Time: 14:00 (Raining). Location: The Dark Woods.
Player Party: 
- Elara (Level 4 Mage, 12/20 HP)
- Gorn (Level 4 Fighter, 30/30 HP)

Recent Events:
Player attacked Goblin with Sword. Dice: 18 vs AC 14. HIT for 6 damage.

INSTRUCTION: Narrate the outcome of this hit and dictate what the Goblin does next on its turn. Do not play for the user.]
```

### Debugging Value
If the LLM is consistently ignoring the time of day, or inventing character hits that didn't happen, you check this file. You `console.log()` the output string to ensure the ` combat.service` side-effects were actually correctly concatenated into the "Recent Events" block. If they aren't there, the bug is in the runner pushing context, not the LLM malfunctioning!
