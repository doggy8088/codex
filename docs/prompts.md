## Custom Prompts

將經常使用的提示儲存為 Markdown 檔案，為 codex 提供額外的指示和指導，並從斜線選單快速重複使用它們。

- Location: Put files in `$CODEX_HOME/prompts/` (defaults to `~/.codex/prompts/`).
- File type: Only Markdown files with the `.md` extension are recognized.
- Name: The filename without the `.md` extension becomes the slash entry. For a file named `my-prompt.md`, type `/my-prompt`.
- 內容：當您在斜線彈出視窗中選擇項目並按 Enter 時，檔案內容將作為您的訊息傳送。
- 如何使用：
  - 開始新會話（codex 在會話開始時載入自訂提示）。
  - In the composer, type `/` to open the slash popup and begin typing your prompt name.
  - Use Up/Down to select it. Press Enter to submit its contents, or Tab to autocomplete the name.
- Notes:
  - Files with names that collide with built‑in commands (e.g. `/init`) are ignored and won’t appear.
  - 新的或已變更的檔案會在會話開始時被發現。如果您在 codex 運行時新增新提示，請開始新會話以載入它。
