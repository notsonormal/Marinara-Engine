# Master Index: Marinara Engine Documentation

Welcome to the root index of the `explaining_docs` directory! If you are looking for a specific explainer, trace, or architectural document, use this table of contents.

For an explanation on *how* to use this folder and the underlying structural methodologies (like Divio and C4), please read the [Documentation Playbook (README)](README.md).

---

## 🏗️ 1. Architecture (C4 Model)
High-level visual maps of the system boundaries and logic blocks.
* [System Context (`context.md`)](architecture/context.md)
* [System Container (`container.md`)](architecture/container.md)
* [System Component (`component.md`)](architecture/component.md)

## 🗺️ 2. Structure (Codebase Archaeology)
Information regarding the exact data flow and the physical state of the codebase files.
* [Packages Breakdown (`packages.md`)](structure/packages.md)
* [Database Schema ERD (`database_schema.md`)](structure/database_schema.md)
* [Turn Data Flow (`data_flow.md`)](structure/data_flow.md)

## 📚 3. Core Guides (Divio System)
Goal-oriented and understanding-oriented documentation.

### Tutorials
*Learning-oriented step-by-step concepts.*
* [Tutorials Index](guides/tutorials/index.md)

### How-To Guides
*Task-oriented recipes for achieving specific results.*
* [How to Create an Agent](guides/how-to/create-an-agent.md)
* [How to Create a CSS Theme](guides/how-to/create-a-theme.md)
* [How-To Index](guides/how-to/index.md)

### Reference
*Information-oriented specs and lists.*
* [Built-in Agents List](guides/reference/agents.md)
* [REST API Endpoint Overview](guides/reference/api.md)
* [Reference Index](guides/reference/index.md)

### Explanation (Deep Dives)
*Understanding-oriented details on engine internals.*
* [Engine Modes Explained](guides/explanation/modes.md)
* [Frontend State Management (Zustand vs React Query)](guides/explanation/state-management.md)
* [The Expression System (Sprites & Animations)](guides/explanation/expression_system.md)
* [AI Image Generation Architecture](guides/explanation/image_generation.md)

**Execution Traces & Deep Dives**
* [Game Mode Execution Trace](guides/explanation/game_mode/execution_trace_turn.md)
* [Game Mode State Map](guides/explanation/game_mode/state_map.md)
* [Game Mode API Contract (GM Prompts)](guides/explanation/game_mode/gm_prompts_contract.md)
* [Game Mode Known Gotchas](guides/explanation/game_mode/gotchas.md)
* [Conversation Mode Execution Trace](guides/explanation/conversation_mode/execution_trace.md)
* [Roleplay Mode Execution Trace](guides/explanation/roleplay_mode/execution_trace.md)

* [Explanation Index](guides/explanation/index.md)

## 🏛️ 4. Architecture Decision Records (ADRs)
The permanent record of why technology was chosen over alternatives.
* [ADR 0001: Use SQLite for Local Persistence](adr/0001-use-sqlite.md)
* [ADR 0002: Use React & Tailwind CSS v4 for Client](adr/0002-react-tailwind.md)
