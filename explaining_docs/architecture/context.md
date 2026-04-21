# C4 Level 1: System Context Diagram

The System Context diagram is a high-level view showing the Marinara Engine and how it interacts with users and external systems.

## Diagram

```mermaid
C4Context
  title System Context diagram for Marinara Engine
  
  Person(user, "Player/User", "A user interacting with the engine to play games, chat, or roleplay.")
  System(marinara, "Marinara Engine", "A local, AI-powered chat, roleplay, and game engine handling conversations, lore, UI, and logic.")
  
  System_Ext(llm, "LLM Providers", "OpenAI, Anthropic, Google, OpenRouter, Mistral, Cohere, etc. Provides text generation.")
  System_Ext(image_gen, "Image Gen Providers", "Stability AI, Pollinations, NovelAI, ComfyUI, AUTOMATIC1111. Provides image/sprite generation.")
  System_Ext(chub_ai, "Chub.ai", "Platform for discovering and downloading character cards/bots.")
  System_Ext(spotify, "Spotify", "Provides music playback and atmospheric control for scenes.")
  System_Ext(buttplug, "Buttplug.io / Intiface", "Provides haptic device control for immersive hardware.")
  System_Ext(giphy, "Giphy API", "Provides GIF search and embedding.")

  Rel(user, marinara, "Plays via Web Browser or Mobile App")
  Rel(marinara, llm, "Sends prompts, receives AI responses")
  Rel(marinara, image_gen, "Sends image prompts, receives generated visuals")
  Rel(marinara, chub_ai, "Searches and downloads character cards")
  Rel(marinara, spotify, "Controls playback state and playlist selection")
  Rel(marinara, buttplug, "Sends haptic commands")
  Rel(marinara, giphy, "Searches for GIFs")
```

## Elements

*   **Player/User**: The primary actor using the engine via a browser, PWA, or WebView wrap.
*   **Marinara Engine**: The core web app running locally (on Docker, Windows, macOS/Linux, or Android/Termux).
*   **LLM Providers**: The brain of the characters and agents, using user-provided API keys or local configurations.
*   **Image Gen Providers**: The source of dynamic character avatars, background images, and scene visuals.
*   **Chub.ai**: A major community hub directly integrated into the app for pulling content.
*   **Spotify**: Integrated directly for scene-based music control via the "Spotify DJ" agent.
*   **Buttplug.io**: Integrated for haptic hardware via the "Love Toys Control" agent.
*   **Giphy API**: Used for searching and inserting animated images.
