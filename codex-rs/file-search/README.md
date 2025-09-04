# codex_file_search

Codex 的快速模糊檔案搜尋工具。

底層使用 <https://crates.io/crates/ignore>（`ripgrep` 採用的函式庫）遍歷目錄（遵循 `.gitignore` 等規則）以產生可搜尋的檔案清單，接著以 <https://crates.io/crates/nucleo-matcher> 對使用者提供的 `PATTERN` 進行模糊比對。
