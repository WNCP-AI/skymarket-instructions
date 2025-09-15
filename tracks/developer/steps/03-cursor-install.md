 i# Step 3: Cursor — Quick Start

Get Cursor installed and productive fast. Covers setup, models, agent modes, and core functionality.

## Objective

Install Cursor, pick the right model, know the agent modes, and use the features that matter.

## 1) Setup

### Install

```bash
# macOS
brew install --cask cursor
```

```bash
# Windows
winget install Cursor.Cursor
```

```bash
# Linux (download)
# Grab the latest from the website if your package manager doesn’t have Cursor.
# https://cursor.com
```

### First run

- Sign in with GitHub/Google.
- Toggle AI pane: `Cmd/Ctrl + L`.
- Open Cursor Settings: `Cmd/Ctrl + Shift + J` (or Command Palette).

### Optional but useful

- Enable Long Context Chat: Settings → Beta → Long Context Chat. Toggle chat modes with `Cmd/Ctrl + .`.
- Add API keys if you want to use your own quotas/models: Settings → Models → OpenAI / Anthropic / Google / Azure → Verify.

```plaintext
# .cursorignore (same syntax as .gitignore)
dist/
*.log
config.json
```

```plaintext
# .cursorrules (project-specific AI rules)
# e.g., architecture, style, review focus
```

## 2) Models

- Select a model from the dropdown in Chat or with `Cmd/Ctrl + K`.
- Good defaults: GPT‑5, Claud 4 Sonnet, `cursor-small` (fast, unlimited).
- Add custom model names: Settings → Models → Model Names.
- Long‑context examples: `gpt-5`, `gemini-1.5-flash-500k`, `claude-4 sonnet 200k`.
- Context limits (practical): Chat ~20k tokens; Cmd‑K ~10k; Long Context uses model max.

## 3) Agent modes

- Chat (AI pane): `Cmd/Ctrl + L`. Use `@file` mentions. Long Context toggles with `Cmd/Ctrl + .`.
- Inline Cmd‑K: `Cmd/Ctrl + K` for precise, file‑scoped edits. Uses chunking strategies (auto, full file, outline, chunks).
- Tab (autocomplete): ghost text for inserts; diff popups for modifications.
  - Accept `Tab`, reject `Esc`, partial accept next word `Cmd/Ctrl + →`.
  - Cursor predicts next edit locations; keep tapping `Tab` to step through.
- AI Review: review Working State, Diff with main, or Last commit.

## 4) Core functionality

- Apply code from Chat: click ▶ in a chat code block to edit files.
  - Accept `Cmd/Ctrl + Enter`, reject `Cmd/Ctrl + Backspace` in the diff.
- AI Fix linter errors: hover error → AI fix, or `Cmd/Ctrl + Shift + E`.
- Toggle AI pane: `Cmd/Ctrl + L`. Chat history: AI pane button or `Cmd/Ctrl + Alt/Option + L`.
- Peek views + Tab: works in definition peek (great for propagating signature changes). Vim: `gd` then accept Cursor edits.
- Chat settings: Settings → Features → Chat (Always search web, Default to no context, Auto scroll, etc.).
- Corporate proxy edge case: disable HTTP/2 in settings JSON.

```json
{
  "cursor.general.disableHttp2": true
}
```

## 5) Quick smoke test

```bash
mkdir cursor-quickstart && cd $_
printf "console.log('Hello from Cursor!')\n" > test.js
cursor .
```

1. Chat: `Cmd/Ctrl + L` → ask “Explain test.js” → add `@test.js`.
2. Cmd‑K: open `test.js` → `Cmd/Ctrl + K` → “add try/catch logging”.
3. Tab: type a function signature and accept suggestions with `Tab`.

## 6) Shortcuts you’ll actually use

```plaintext
AI Chat: Cmd/Ctrl + L
Inline: Cmd/Ctrl + K
Partial accept next word: Cmd/Ctrl + →
Accept/Reject applied edit: Cmd/Ctrl + Enter / Cmd/Ctrl + Backspace
Toggle chat modes (Long Context): Cmd/Ctrl + .
Command Palette: Cmd/Ctrl + Shift + P
```

## 7) Minimal troubleshooting

- AI not responding: sign out/in, check usage/plan, confirm network, try another model.
- Context too big/slow: reduce `@file`s; use Cmd‑K (chunks) or Long Context.
- Corporate proxy issues: set `cursor.general.disableHttp2` as above.
- Extensions flaky: Command Palette → “Reload Window”.

Next: set up Context7 so Cursor can pull official docs in‑chat.
