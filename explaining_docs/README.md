# Explaining Docs: The Documentation Playbook

This folder contains the systematic documentation for **Marinara Engine**. Unlike standard developer setup guides, this directory is specifically structured to break down the architectural complexity of the engine and serve as the foundation for deep-dive investigation, troubleshooting, and architectural tracking.

## Methodologies Used

We utilize four distinct methodologies to categorize information properly, preventing the documentation from becoming a single unreadable mass.

### 1. The C4 Architecture Model (`/architecture`)
Provides visual, zoomable layers of the engine.
- **Context:** High-level interactions with users and 3rd-party APIs (Spotify, Chub.ai, LLMs).
- **Container:** The physical units of deployment (Client, Server API, SQLite).
- **Component:** The logical blocks within the containers (e.g., Agent Runner, Prompt Pipeline, Frontend Routers).

### 2. Codebase Archaeology (`/structure`)
Empirical mappings of how data moves and lives at rest.
- Includes data flow sequence diagrams and database ORM schema structures.

### 3. Divio Documentation System (`/guides`)
Divides all written guides into four distinct quadrants:
1. **Tutorials (Learning-oriented):** Hand-holding for beginners.
2. **How-To Guides (Task-oriented):** Focused recipes to achieve specific outcomes (e.g., *How to create an Agent*).
3. **Reference (Info-oriented):** Dry, technical specs (e.g., *API routes, Agent specs*).
4. **Explanation (Understanding-oriented):** The *why* behind abstract concepts (e.g., *Understanding State Management*).

### 4. Architecture Decision Records (`/adr`)
A running ledger of major technological choices (e.g., *Why use SQLite? Why use Tailwind v4?*). Before swapping major libraries or systems, write an ADR.

---

## Strategy for Future "Deep Dive" Documentation

As we move toward troubleshooting distinct bugs or refactoring complex systems (like Game Mode), we will expand this documentation using the following **Deep Dive Strategies**. 

When investigating a feature or looking for bugs, use these specific methodologies rather than writing generic summaries:

### A. Execution Tracing (The "Path of a Request")
**Purpose:** Maps the exact file paths and functions involved in a chain reaction.
- *Format:* Start at the UI trigger event -> name the UI component file -> name the Fastify route -> name the service functions -> name the DB write.
- *Why:* When a process fails silently, you know exactly which 4 files to check sequentially.

### B. State & Mutability Maps
**Purpose:** Defines who owns what data and who is allowed to change it.
- *Format:* Separate concerns between *Synchronized Server State* (React Query / DB) and *Ephemeral UI State* (Zustand). Map which React components are subscribing to which state stores.
- *Why:* Prevents race conditions and quickly diagnoses visual stuttering/glitches. 

### C. Subsystem API Contracts (The "Black Box")
**Purpose:** Defines strict Inputs/Outputs for complex modular systems (like the Prompt Pipeline).
- *Format:* Define exactly what context variables are pushed in, the order of concatenation, and what string comes out to the LLM Provider.
- *Why:* Essential for debugging LLM behavioral issues (e.g., LLM forgetting things or ignoring system prompts).

### D. Known Gotchas & Bug Vectors (The Mortuary)
**Purpose:** A curated list of fragile, time-sensitive, or hardware-dependent code blocks.
- *Format:* Document race-condition windows in SQLite, timeout issues with local generation (AUTOMATIC1111), and third-party API rate limits.
- *Why:* Creates a triage checklist when users report weird edge-case bugs.
