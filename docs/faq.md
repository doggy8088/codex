## 常見問答

### OpenAI 在 2021 年發布了一個名為 Codex 的模型 - 這有關聯嗎？

在 2021 年，OpenAI 發布了 Codex，一個旨在從自然語言提示產生程式碼的 AI 系統。最初的 Codex 模型已於 2023 年 3 月棄用，與此 CLI 工具無關。

### 支援哪些模型？

我們建議將 Codex 與 GPT-5（我們最好的程式設計模型）搭配使用。預設的推理等級為中等，您可以使用 `/model` 指令為複雜任務升級至高等級。

您也可以透過 API 驗證使用舊版模型，並使用 `--model` 旗標啟動 codex。

### 為何 `o3` 或 `o4-mini` 對我無效？

您的 [API 帳戶可能需要驗證](https://help.openai.com/en/articles/10910291-api-organization-verification)，才能開始串流回應並查看 API 的思路鏈摘要。如果您仍遇到問題，請告知我們！

### 如何阻止 Codex 編輯我的檔案？

預設情況下，Codex 可以修改您目前工作目錄中的檔案（自動模式）。若要防止編輯，請使用 CLI 旗標 `--sandbox read-only` 以唯讀模式執行 `codex`。或者，您可以在對話中途使用 `/approvals` 變更核准層級。

### 它能在 Windows 上運作嗎？

直接在 Windows 上執行 Codex 可能會成功，但並非官方支援。我們建議使用 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install)。 