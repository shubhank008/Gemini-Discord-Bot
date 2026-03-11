# Copilot Instructions for Gemini Discord Bot

## Project Overview

This is a Discord bot powered by Google Gemini AI, built with Node.js (ESM) and `discord.js` v14. It supports conversational AI, multimodal file understanding (images, video, audio, PDFs, Office documents), custom personalities, server/channel chat history, slash commands, admin controls, and several built-in AI tools.

## Technology Stack

- **Runtime**: Node.js v20+ with native ES Modules (`"type": "module"` in `package.json`)

- **Discord library**: `discord.js` 14
- **AI backend**: `@google/genai` (Google Gemini)
- **Key utilities**: `axios`, `sharp`, `office-text-extractor`, `node-os-utils`, `dotenv`
- **Start command**: `npm start` → `node index.js`

## Repository Structure

```
index.js              # Entry point – initializes and logs in the bot
botManager.js         # Discord client, Gemini client, and persisted runtime state
commands.js           # Slash-command definitions
config.js             # User-facing defaults (model, personality, colours, activities)
src/
  bootstrap.js        # Registers Discord lifecycle event handlers
  constants.js        # Internal constants (file size limits, supported MIME types, etc.)
  handlers/
    messageHandler.js      # Processes incoming Discord messages
    interactionHandler.js  # Processes slash commands and button interactions
  services/
    attachmentService.js   # Downloads and extracts text/media from attachments
    conversationService.js # Orchestrates Gemini API calls and chat history
  ui/
    *.js                   # Reusable Discord embeds, buttons, and settings views
  utils/
    *.js                   # Shared Discord helpers (reply, edit, defer, etc.)
tools/                # Optional Gemini function-calling tool definitions
config/               # Runtime-persisted JSON (chat history, user settings, blacklists)
example.env           # Template – copy to .env and fill in secrets
```

## Code Conventions

- **All files use ES Module syntax** (`import`/`export`). Never use `require()`.
- Use `async/await` throughout; avoid raw `.then()` chains.
- Exported state lives in `botManager.js`; import from there rather than creating new globals.
- Configuration tuneable by users belongs in `config.js`; internal constants go in `src/constants.js`.
- Discord interaction helpers (defer, reply, followUp, etc.) belong in `src/utils/`.
- Keep handler files focused: `messageHandler.js` routes messages, `interactionHandler.js` routes interactions — heavy logic goes into services or utils.

## Environment Variables

Required in `.env` (never commit this file):

```
DISCORD_BOT_TOKEN=your_discord_bot_token
GOOGLE_API_KEY=your_google_api_key
```

Optional:
```
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_API_KEY=your_openai_api_key
```

## Running the Bot

```bash
npm install
cp example.env .env   # then fill in your tokens
npm start
```

There is no build step — the bot runs directly from source.

## Testing & Linting

The project currently has no automated test suite or linter configured. When adding new functionality:

- Manually test with a real Discord server and valid API keys.
- Prefer small, isolated changes that are easy to verify end-to-end.
- If adding a test framework in the future, prefer `node:test` (built-in) to keep dependencies minimal.

## Key Patterns to Follow

1. **New slash commands** – Add the definition to `commands.js` and handle the interaction in `interactionHandler.js`. Re-deploy commands after any change.
2. **New Gemini tools** – Add a tool descriptor under `tools/` and register it in `conversationService.js`.
3. **New file types** – Update `src/constants.js` (MIME type lists) and `src/services/attachmentService.js`.
4. **Persisted settings** – Follow the existing pattern in `botManager.js` (load on startup, save on change, read from the `config/` directory).
5. **Error handling** – Catch errors at the service boundary; surface user-friendly messages via Discord reply/followUp; log full errors to the console.

## Security Notes

- Never commit `.env` or any file containing API keys or tokens.
- The `config/` directory contains user data; treat it as sensitive. It is already in `.gitignore`.
- Admin-only commands must check `interaction.member.permissions` before executing.
- Validate and sanitise any user-supplied text before including it in prompts or filenames.
