---
name: viagen-install
description: >
  Install viagen into the current Vite project. Use when the user wants to add
  viagen (AI-powered Vite dev server plugin) to their app. Handles npm
  installation and vite.config update.
allowed-tools: Bash, Read, Edit, Grep, Glob, Write
---

# viagen Install Skill

You are helping the user add **viagen** to their Vite project.

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

### 3. Next steps

Tell the user to run `npx viagen setup` to configure authentication and
environment variables, then `npm run dev` to start.

## Notes

- viagen requires `vite >= 4` as a peer dependency
- Do NOT modify `.env` or run the setup wizard — that's a separate step
