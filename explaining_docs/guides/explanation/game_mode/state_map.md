# Deep Dive: State Map of Game Mode

Game Mode relies heavily on a complex UI featuring numerous sidebars, indicators, and overlays. Debugging visual desyncs (e.g., an inventory not updating after finding an item) requires understanding where the components are drawing their truth from.

## Synchronized Server State (React Query)
The ultimate source of truth for all quantitative game stats is the Backend database, accessed via React Query mutations.

**The Golden Rule of Game Mode Mutability:** 
*The Client UI is NEVER allowed to mathematically mutate game states (like subtracting HP or applying debuffs).* The Client can only *request* an action via the API. The Server performs the math and returns the updated state payload to overwrite the cache.

### Bound Components:
* `GamePartySidebar.tsx`: Reads from the `useSession` cache to render HP/MP bars and active debuffs.
* `GameInventory.tsx`: Reads the serialized item array from the React Query response to list items.
* `GameJournal.tsx`: Renders the active Quests and Lorebook findings parsed from the DB.

## Ephemeral UI State (Zustand)
Game Mode has two primary Zustand stores controlling what you see on the screen without persisting data to SQLite.

### 1. `useGameModeStore` (`game-mode.store.ts`)
Manages structural UI layout parameters for the mode.
* **Controlled States:** 
  * "Is the map overlay currently open?" 
  * "Are we looking at our inventory or our stats page in the sidebar?"
* **Mutability:** Any component can safely update these boolean/enum toggles via `toggleOverlay()` dispatch functions.

### 2. `useGameStateStore` (`game-state.store.ts`)
Manages transient contextual selection during gameplay.
* **Controlled States:**
  * "Which enemy target did the user click on to cast a fireball at?"
  * "Which choice card is currently hovering?"
* **Mutability:** When a user selects a target in `GameCombatUI.tsx`, this store holds that Target ID so that when they finally click "Send" in `GameInput.tsx`, the input component can attach that ID to the API request payload. Once the API call resolves, this store state is usually cleared or reset.
