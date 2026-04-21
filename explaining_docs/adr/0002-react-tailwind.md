# ADR 0002: Use React & Tailwind CSS v4 for Client

## Status
Accepted

## Context
Marinara Engine requires a heavy, highly interactive user interface. It needs to support real-time chat streaming, dynamic animations for sprites, complex settings menus, and draggable modal windows. The ecosystem selected needs to be mature and have a deep library of accessible components. It must also support a "Theme" system to allow switching between UI aesthetics (e.g., modern "Y2K" vs "Classic SillyTavern").

## Decision
We will build the Client package (`packages/client`) using **React** and style it using **Tailwind CSS v4**.

## Consequences
**Positive:**
- React's ecosystem is unmatched. We can easily utilize Framer Motion for sprite animations and React Query for asynchronous data fetching.
- Tailwind v4 provides a lightning-fast utility-first class system that avoids the need to write custom CSS files for every component.
- Implementing dark mode and themes is straightforward using CSS variables injected globally and consumed by Tailwind utilities.

**Negative:**
- React has a steep learning curve and requires strict state management discipline (hence the introduction of Zustand).
- Tailwind classes can make standard React components look bloated (`className="flex flex-col items-center justify-center p-4..."`), requiring developers to extract common patterns into reusable abstraction components.
