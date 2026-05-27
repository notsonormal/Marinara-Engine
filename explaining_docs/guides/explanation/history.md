# Explanation: History, Swipes, and Snapshots

In the Marinara Engine, chat history is more than just a flat list of messages. It supports branching realities (swipes), structured state tracking (snapshots), and robust cascading deletes to manage complex roleplay and game sessions.

## 1. Messages and Swipes

The core conversational data model is split into two tables: `messages` and `message_swipes`.

### Messages
A `message` represents a discrete conversational turn in a `chat`. It defines the metadata for that turn:
- **Role:** Who sent it (`user`, `assistant`, `system`, `narrator`).
- **Character ID:** Which character generated the response.
- **Active Swipe Index:** A pointer indicating *which* version of this message is currently selected by the user.

### Swipes (`message_swipes`)
To support re-rolling ("swiping") AI generations without losing previous attempts, the engine stores the actual text content in the `message_swipes` table.
- Each `message_swipe` is linked to a parent `messageId` and is assigned an `index` (0, 1, 2, etc.).
- When a user asks the AI to regenerate a response, a new `message_swipe` is created with the next available index.
- When the user navigates left or right in the UI, the `activeSwipeIndex` on the parent `message` is updated, and the client renders the text from the corresponding swipe.

> [!NOTE]
> Even user messages technically use this architecture, meaning users can edit their own past messages without deleting them, creating "swipes" of their own inputs.

## 2. Snapshots and Checkpoints

In Game Mode and Roleplay Mode, the engine needs to track the state of the world at any given point in history. This is managed via Snapshots and Checkpoints.

### Game State Snapshots (`game_state_snapshots`)
A snapshot captures the exact state of the world (time, weather, location, present characters, player stats) for a specific turn.
- Snapshots are tied to a specific `messageId` and `swipeIndex`. 
- Because each swipe can branch the story differently, the engine tracks game state *per swipe*, not just per message. If Swipe 0 results in the player traveling to the forest, and Swipe 1 results in them going to the tavern, the snapshots will reflect those divergent realities.

### Checkpoints (`game_checkpoints`)
Checkpoints act as "save states" or bookmarks in the history.
- They are linked to a specific `snapshotId` and `messageId`.
- They can be triggered manually by the user (e.g., "Before Boss Fight") or automatically (e.g., "Session Start", "Location Change").
- Checkpoints allow the system to quickly retrieve denormalized context without crawling the entire chat history.

## 3. History Deletion

The engine handles history deletion safely using database-level cascading rules.

### Deleting Messages
When you delete a message (or use "Delete from here" to truncate the history), the engine deletes the record(s) from the `messages` table. 

Because the database schema is configured with `onDelete: "cascade"`, deleting a message automatically and safely cleans up all attached dependencies:
- All related `message_swipes` are deleted.
- All related `game_state_snapshots` attached to that message are orphaned or cascade-deleted (handled at the application/DB level).
- All related `game_checkpoints` tied to that message are removed.

This ensures that truncating history does not leave ghost swipes or corrupted game states behind in the database.
