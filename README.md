# 15 Puzzle – Mobile-First PWA Game

A modern, mobile-first implementation of the classic 15 Puzzle built with SvelteKit and pure CSS.

Designed as a Progressive Web App (PWA) with a game-like UI optimized primarily for mobile devices while maintaining a clean desktop experience.

---

## Tech Stack

- SvelteKit (Svelte 5 runes)
- Vanilla CSS
- Inter font
- Progressive Web App (PWA)
- No external UI or chart libraries (intentional for performance)

---

## Progressive Web App

The application is installable as a Progressive Web App:

- Web manifest configured
- Service worker registered
- App shell cached
- Standalone mobile experience

This improves perceived performance and provides an app-like experience on supported devices.

---

## Project Structure

```text
.
├── package.json
├── vite.config.js
├── svelte.config.js
└── src/
    ├── routes/
    │   ├── +layout.svelte
    │   └── +page.svelte
    ├── lib/
    │   └── styles/
    |    ├── tokens.css
    │    └── global.css
    └── assets/
        └── favicon.svg
```
---

## Getting Started

Install dependencies:

```bash
npm install
```

Run locally:

```bash
npm run dev
```

Build:

```bash
npm run build
```

Preview production build:

```bash
npm run preview
```

Open the app in your browser:

http://localhost:5173

---
