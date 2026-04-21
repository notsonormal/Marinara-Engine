# Packages Directory Breakdown

Marinara Engine is set up as a pnpm monorepo (or workspace) where code is split into three main packages to cleanly separate frontend, backend, and shared logic.

## 1. `packages/client`

This is the frontend of the application, responsible for rendering the UI and maintaining client-side state.

**Key Technologies:**
- **React 19:** The core UI framework.
- **Tailwind CSS v4:** Utility-first styling framework for UI elements.
- **Vite 6:** The build tool and dev server.
- **Framer Motion:** Handles complex animations like panning/zooming avatars and smooth transitions.
- **Zustand:** A lightweight state management library for session data.
- **React Query:** Manages fetching, caching, synchronizing, and updating server data.

**Key Responsibilities:**
- Rendering the user interfaces for Conversation mode, Roleplay mode, Game mode, and Settings.
- Handling responsive views across desktop and mobile.
- Rendering dynamic effects like character expression sprites, background scenes, and weather overlays.

## 2. `packages/server`

This is the backend API and execution engine. It's the most complex package, as it handles AI reasoning, database interactions, and local hardware connections.

**Key Technologies:**
- **Node.js:** The runtime environment.
- **Fastify 5:** A fast and low-overhead web framework for routing an API.
- **SQLite:** The file-based database for local, offline storage.
- **Drizzle ORM:** A type-safe ORM for interacting with SQLite.

**Core Directories in `src/`:**
- **`routes/`**: Fastify HTTP endpoints, acting as controllers that accept requests from the Client App.
- **`db/`**: Drizzle schemas, migrations, and database connection logic.
- **`services/`**: The business logic of the engine.
  - **`agents/`**: Implementations for the 25+ AI agents (World State, Prompt Reviewer, Quest Tracker, etc.).
  - **`llm/`**: Connection managers for sending payloads to various providers (OpenAI, Anthropic, OpenRouter, etc.).
  - **`prompt/`**: The assembly pipeline that handles complex macro replacements, chunking, and final formatting of the context window.
  - **`lorebook/`**: Retrieval logic for determining which lore entries to inject based on user chat history.
  - **`image/`**: Orchestration logic for building and requesting image prompts.

## 3. `packages/shared`

This package bridges the gap between the Client and Server, ensuring that both ends agree on data formats without having to write duplicate logic.

**Key Technologies:**
- **TypeScript 5:** For strict typing.
- **Zod:** A schema declaration and validation library.

**Core Directories in `src/`:**
- **`schemas/`**: Zod domains. e.g., validation rules for a Character or a Chat branch. Ensure the Server receives valid API bodies.
- **`types/`**: Raw TS interfaces representing domain objects. Includes generic, application-wide types.
- **`constants/`**: Hardcoded constants (such as version strings, default models, or common enums) shared in the client components and server logic.
- **`utils/`**: Generic formatting or helper functions that don't depend on DOM (Client) or Node (Server) APIs.
