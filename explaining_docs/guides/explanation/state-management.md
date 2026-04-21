# Explanation: Frontend State Management

In the `packages/client` architecture, state management is strictly isolated into two categories: **Ephemeral UI State** and **Synchronized Server State**. 

## Synchronized Server State (React Query)
Any data that lives permanently in the SQLite database is managed by **@tanstack/react-query**.

**Examples:**
- A list of Characters.
- The message history of a Chat.
- A user's active Lorebooks.

**Why:** React Query handles boilerplate around fetching, caching, retrying on failure, and polling. 
- When a user submits a chat message, we use a React Query `useMutation`.
- We immediately execute an `onMutate` optimistic update to insert the user's text on the screen instantly, making the UI feel snappy even if the Server API takes a second to process.

## Ephemeral UI State (Zustand)
Any data that is discarded when the tab closes, or is purely aesthetic, is managed by **Zustand** stores located in `packages/client/src/stores/`.

**Examples:**
- `useSettingsStore`: Is the user's sidebar currently collapsed? Is Dark Mode forced on?
- `useChatStore`: What is the currently active chat ID? Are we dragging to zoom a sprite right now?

**Why:** Zustand is significantly lighter and less opinionated than Redux. It allows our React components to cleanly subscribe to *only* the specific slices of state they care about, preventing massive re-render waterfalls across the Client App.

## Rule of Thumb for Contributors
1. If you need to **save** it to the hard drive: use Fastify + React Query.
2. If you need to **remember** it while jumping between routes (but don't care if a page refresh kills it): use Zustand.
3. If only a single button/modal cares about it: use a local React `useState`.
