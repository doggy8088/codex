## Custom Prompts

Save frequently used prompts 作為 Markdown 檔案為 codex 提供額外的指示和指導 和 reuse them quickly 來自 the slash menu.

- Location: Put files in `$CODEX_HOME/prompts/` (defaults to `~/.codex/prompts/`).
- File type: Only Markdown files with the `.md` extension are recognized.
- Name: The filename without the `.md` extension becomes the slash entry. For a file named `my-prompt.md`, type `/my-prompt`.
- Content: The file contents are sent 作為 您的 message 當 您 select the item 在 the slash popup 和 press Enter.
- 如何 到 use:
  - Start a new session (codex loads custom prompts 在 session start).
  - In the composer, type `/` to open the slash popup and begin typing your prompt name.
  - Use Up/Down to select it. Press Enter to submit its contents, or Tab to autocomplete the name.
- Notes:
  - Files with names that collide with built‑in commands (e.g. `/init`) are ignored and won’t appear.
  - New 或 changed 檔案為 codex 提供額外的指示和指導 are discovered 在 session start. 如果 您 add a new prompt while codex is running, start a new session 到 pick 它 up.
