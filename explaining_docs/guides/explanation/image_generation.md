# Explanation: AI Image Generation 

While the **Expression System** relies on ultra-fast, pre-rendered static sprites, Marinara Engine still heavily utilizes dynamic AI Image Generation (via providers like Stability AI, NovelAI, ComfyUI, etc.) for environmental and contextual illustrations.

Because generating an image takes significantly longer than generating text (and costs more compute), the engine uses specialized Agents to orchestrate *when* and *why* an image is generated, so it doesn't break the flow of the game.

## Where Image Generation is Used

### 1. "Selfies" and In-Chat Photos (Conversation Mode)
In the Messenger-style Conversation mode, characters act like real people texting from a phone.
- **The Trigger:** The **Illustrator Agent** analyzes the character's generated text. If the character says something highly visual—like *"Check out the new dress I just bought!"* or *"I just baked these cookies!"*—the Agent flags the message.
- **The Execution:** The Agent quietly builds an image prompt behind the scenes (e.g., `A photo taken from a smartphone of a woman wearing a red dress, selfie angle`) and requests an image from the connected provider.
- **The Result:** The image is attached to the chat bubble just like a real photo attachment in Discord or iMessage.

### 2. Dynamic Background Scenes (Roleplay & Game Modes)
When playing in immersive visual modes, the scene should reflect the narrative.
- **The Trigger:** When a new `Chat` or `Game Session` begins, or when the **Background Agent** detects a major location change in the narrative (e.g., *"We leave the tavern and step out into the dark, rainy streets"*).
- **The Execution:** The system generates a wide-aspect-ratio (usually 16:9) cinematic background illustrating the new location.
- **The Result:** The UI's backdrop gracefully cross-fades into the newly generated image, anchoring the character sprites in a new environment.

### 3. Game Assets & Items (Game Mode)
In Game Mode, users collect items and visit distinct nodes.
- **The Trigger:** When the AI Game Master grants the player a new unique item (e.g., *The Sword of 1,000 Truths*), or reveals a new enemy encounter.
- **The Execution:** A background generation request is fired off specifically tailored to item or portrait dimensions.
- **The Result:** The `GameInventory.tsx` UI populates with a uniquely generated icon for that specific item, rather than relying on generic stock icons.

## How it Works Under the Hood

The Image Generation pipeline separates the *decision* to generate an image from the *act* of generating it. 

1. **The LLM is NOT the Image Generator:** The LLM powering the character (like Claude 3 or Llama 3) does not generate images directly. It simply generates the prose.
2. **The Output Analysis:** Post-processing agents (like `Illustrator` or `Background`) scan that prose to determine if a picture is warranted.
3. **Prompt Engineering the Image:** If an image is needed, the Agent looks at the character's physical description tags (e.g. `blonde hair, blue eyes, wearing armor`) and programmatically combines them with the context (e.g. `holding a sword`) to build a highly-optimized prompt for Stable Diffusion or Midjourney.
4. **Asynchronous Fetching:** The request is sent to the Image Provider entirely decoupled from the text response. This means the user reads the text immediately, and a few seconds later, the image seamlessly pops into the UI once it arrives.
