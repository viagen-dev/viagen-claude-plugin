---
name: viagen-status
description: >
  Check viagen installation and configuration status. Use when the user wants to
  verify their viagen setup is working, check API key status, or diagnose issues.
allowed-tools: Bash, Read, Grep, Glob
---

# viagen Status Skill

You are helping the user verify their **viagen** installation and configuration.

## Steps

### 1. Check installation

Verify `viagen` is in devDependencies:
```bash
npm ls viagen
```

### 2. Check vite config

Find the vite config file and confirm `viagen()` is in the plugins array:
- Look for `vite.config.ts`, `vite.config.js`, or `vite.config.mts`

### 3. Check environment

Read `.env` and verify the required variables are set (do NOT print secret values,
just confirm they exist):
- `ANTHROPIC_API_KEY` or `CLAUDE_OAUTH_TOKEN`
- `GITHUB_TOKEN` (optional, for sandbox git operations)
- `VERCEL_TOKEN` (optional, for sandbox deploys)

### 4. Health check (if dev server is running)

If the user has a dev server running, check the health endpoint:
```bash
curl -s http://localhost:5173/via/health
```

### 5. Report

Summarize what's configured and what's missing. If anything is missing,
suggest running `/viagen-setup` to fix it.
