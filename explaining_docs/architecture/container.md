# C4 Level 2: Container Diagram

The Container diagram zooms into the Marinara Engine system to show its major logical deployable units (containers).

## Diagram

```mermaid
C4Container
  title Container diagram for Marinara Engine
  
  Person(user, "Player/User", "A user playing the game or chatting.")
  
  System_Boundary(marinara_engine, "Marinara Engine") {
    Container(spa, "Client App", "React 19, Tailwind v4, Vite", "Provides the UI for conversations, game modes, and settings. Runs in the browser or PWA/WebView.")
    Container(api, "Server API", "Fastify 5, Node.js", "Handles business logic, AI agent orchestration, prompt assembly, and database interactions.")
    ContainerDb(db, "SQLite Database", "SQLite, Drizzle ORM", "Stores chats, characters, presets, lorebooks, and settings locally using file-based storage.")
    Container(shared, "Shared Types", "TypeScript, Zod", "Provides shared type definitions, schemas, and schemas across both Client and Server.")
  }

  System_Ext(llm, "LLM Providers", "OpenAI, Anthropic, etc.")
  System_Ext(image_gen, "Image Gen Providers", "Stability AI, NovelAI, etc.")
  
  Rel(user, spa, "Views and interacts via HTTP/HTTPS")
  Rel(spa, api, "Makes API calls", "JSON/HTTP")
  Rel(api, db, "Reads & Writes", "SQL")
  Rel(spa, shared, "Imports types")
  Rel(api, shared, "Imports types")
  
  Rel(api, llm, "Requests text completions", "HTTP/Sockets")
  Rel(api, image_gen, "Requests image generations", "HTTP")
```

## Containers

1.  **Client App (`packages/client`)**: A single-page application built with React. It uses Zustand for state management and React Query for data fetching. It connects to the server API to manage the UI states for the Conversation, Roleplay, and Game modes.
2.  **Server API (`packages/server`)**: A Fastify Node.js server. This is the heart of the engine. It manages the agent processing pipeline, routing HTTP requests to LLM providers, generating responses, running specialized AI agents (like World State, Quest Tracker, Combat), and reading/writing to the local database.
3.  **SQLite Database**: Embedded local database managed via Drizzle ORM. Keeps track of all historical data, chats, and configurations locally on the user's machine without the need for cloud sync.
4.  **Shared Types (`packages/shared`)**: Contains Zod validators, schemas, and shared constants. Ensures data integrity and type safety across the Client-Server boundary.
