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

### 3. Add the client script (SSR frameworks only)

For plain Vite apps, the chat panel is injected automatically. SSR frameworks
(React Router, Remix, SvelteKit, etc.) render their own HTML, so you need to
add a script tag to the root layout's `<head>`:

```html
<script src="/via/client.js" defer></script>
```

To detect if this is needed, check if the project uses an SSR framework:
- **React Router / Remix** — look for `app/root.tsx`, add the script in the `<head>`
- **SvelteKit** — look for `src/app.html`, add the script in the `<head>`
- **Nuxt** — look for `app.vue` or `nuxt.config.ts`

If the project is a plain Vite SPA (just `index.html`), skip this step.

### 4. Next steps

Tell the user:
- Run `npm run dev` to start the local dev server with viagen
- To use sandboxes via the CLI, run `npx viagen setup` in a separate terminal —
  it's an interactive wizard that configures auth, GitHub, Vercel, and `.env`

## Notes

- viagen requires `vite >= 4` as a peer dependency
- Do NOT modify `.env` or run the setup wizard — it's interactive and must be
  run separately by the user
