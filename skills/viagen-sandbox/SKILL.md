---
name: viagen-sandbox
description: >
  Deploy the current project to a remote Vercel Sandbox. Use when the user wants
  to launch a viagen sandbox — a remote VM-like environment where Claude can
  read, write, and push code.
allowed-tools: Bash, Read, Grep, Glob
---

# viagen Sandbox Skill

You are helping the user deploy their project to a **Vercel Sandbox** via viagen.

## Prerequisites

Before running, confirm:
1. The project has viagen installed (`viagen` in devDependencies)
2. A `.env` file exists with viagen config (run `/viagen-setup` first if not)

## Deploy a sandbox

```bash
npx viagen sandbox
```

This deploys the dev server to an isolated remote environment where Claude can
read, write, and push code.

## Common options

```bash
# Deploy on a specific branch
npx viagen sandbox --branch feature/my-thing

# Set a longer timeout (default: 30 min)
npx viagen sandbox --timeout 60

# Auto-send a prompt on load
npx viagen sandbox --prompt "build me a landing page"

# Stop a running sandbox
npx viagen sandbox stop <sandboxId>
```

## Notes

- Sandboxes last 45 min on Vercel Hobby, 5 hours on Pro
- The sandbox command is interactive — it will print the sandbox URL when ready
- If the user provides flags (branch, timeout, prompt), pass them through
