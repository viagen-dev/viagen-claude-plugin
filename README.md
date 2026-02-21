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

3. Run the setup skill:
   ```
   /viagen-setup
   ```

## What it does

The `/viagen-setup` skill will:

1. Install the `viagen` npm package as a dev dependency
2. Add the `viagen()` plugin to your Vite config
3. Run the interactive setup wizard (`npx viagen setup`)
4. Verify your `.env` is configured correctly

## Requirements

- A project using [Vite](https://vitejs.dev/) >= 4
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI

## License

MIT
