# How to Create and Manage a Theme

Marinara Engine utilizes **Tailwind CSS v4** in combination with CSS Variables to provide a robust, hot-swappable theming engine.

## The Problem
Because Tailwind resolves classes like `bg-red-500` at build time, hardcoding colors (`bg-[#2A2A2A]`) directly into React components makes theming impossible. 

## The Solution
Instead of hardcoded colors, the components in `packages/client` use semantic Tailwind classes powered by CSS Variables.

For example, a component might look like this:
```tsx
<div className="bg-surface text-content border border-border">
  Hello World!
</div>
```
The values for `--color-surface`, `--color-content`, and `--color-border` are injected at the root of the document based on the user's active theme.

## Creating a New Theme

1. **Locate the Global CSS**
   Look for the core stylesheet (likely `globals.css` or `theme.css` inside `packages/client/src/styles/`).
   
2. **Define a New Data Theme Block**
   Create a block that targets a specific wrapper attribute.
   ```css
   [data-theme="cyberpunk"] {
     --color-background: #000000;
     --color-surface: #111111;
     --color-primary: #00ff00;
     --color-content: #cccccc;
     --color-border: #333333;
   }
   ```
   
3. **Update the Client Theme Selector**
   Add your new `cyberpunk` option to the Client's settings dropdown menu. When the user selects it, the top-level `<html>` tag updates to `<html data-theme="cyberpunk">`. 

4. **Tailwind v4 Integration**
   Tailwind v4 natively supports consuming CSS variables like this if they are correctly mapped in the CSS setup block. Be cautious to only use HSL or HEX definitions that match the engine's standard format.
