# viagen Claude Code Plugin

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) marketplace plugin that installs and configures [viagen](https://github.com/viagen/viagen) — an AI-powered Vite dev server plugin.

## Installation

1. Add the marketplace:
   ```
   /plugin marketplace add viagen-dev/viagen-claude-plugin
   ```

2. Install the plugin:
   ```
   /plugin install viagen@viagen-marketplace
   ```

3. Run the install skill:
   ```
   /viagen-install
   ```

## Skills

- `/viagen-install` — Install viagen into your Vite project (npm install + vite.config update)
- `/viagen-sandbox` — Deploy to a remote Vercel Sandbox
- `/viagen-status` — Check viagen installation, config, and health
- `/viagen-setup-auth` — Configure sandbox auth bypass (auto-authenticate in sandbox mode)

## Requirements

- A project using [Vite](https://vitejs.dev/) >= 4
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## License

MIT
