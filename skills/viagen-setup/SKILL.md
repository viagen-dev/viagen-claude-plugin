---
name: viagen-setup
description: >
  Install and set up viagen in the current project. Use when the user wants to
  add viagen (AI-powered Vite dev server plugin) to their project. Handles npm
  installation, vite.config update, and running the setup wizard.
allowed-tools: Bash, Read, Edit, Grep, Glob, Write
---

# viagen Setup Skill

You are helping the user install and configure **viagen** — a Vite plugin that
embeds Claude AI directly into the Vite dev server.

## Steps

### 1. Install the npm package

Run:
```bash
npm install --save-dev viagen
```

### 2. Add viagen to the Vite config

Find the project's `vite.config.ts` (or `.js`/`.mts`) and add the viagen plugin:

```ts
import { viagen } from 'viagen'

export default defineConfig({
  plugins: [
    // ... existing plugins
    viagen(),
  ],
})
```

If the file already imports `viagen`, skip this step.

### 3. Run the setup wizard

Run:
```bash
npx viagen setup
```

This will interactively configure:
- Claude authentication (OAuth or API key)
- GitHub CLI setup
- Git repository detection
- Vercel integration (for sandboxes)
- Environment variables in `.env`

### 4. Verify

Confirm that `.env` was created/updated with the required variables.
Tell the user they can now run `npm run dev` to start.

## Notes

- viagen requires `vite >= 4` as a peer dependency
- The setup wizard is interactive — let the user respond to prompts
- Do NOT overwrite existing `.env` values without asking
