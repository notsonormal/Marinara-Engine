# REST API Reference Overview

The Fastify server (`packages/server`) exposes an API consumed by the Client. All endpoints logically live under `/api/`.

## Chats & Messaging

### `POST /api/chat/:id/turn`
**The most important endpoint in the system.**
- **Purpose:** Submits a user message and triggers the complete Agent Runner and Prompt Pipeline loop.
- **Payload:** Extracted variables from the user input (text, images attached, narrative `/commands`).
- **Response:** The finalized AI response block, along with telemetry data representing tokens used and side-effects (e.g., changes to background images).

### `GET /api/chats/:id`
- **Purpose:** Loads the initial configuration and message history for a specific conversation.

### `PATCH /api/chat/:id/messages/:msg_id`
- **Purpose:** Edit an existing message in the chat timeline (used for correcting AI responses or formatting).

## Characters & Entities

### `GET /api/characters`
- **Purpose:** Retrieves a paginated list of downloaded characters for the Bot Browser.
- **Response:** An array of character metadata (names, tags, avatar URIs).

### `POST /api/characters/import`
- **Purpose:** Accepts a Character Card (V2 spec JSON or PNG metadata) and writes it into the local database, extracting the image safely to the disk assets folder.

## Settings & Connections

### `GET /api/connections`
- **Purpose:** Fetch configured LLM/Image provider credentials (masks API keys for security).

### `POST /api/settings/override`
- **Purpose:** Update global or chat-specific API routing (e.g., swapping from OpenAI GPT-4o to a Local Ollama instance mid-chat).
