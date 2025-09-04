## 常見問題

### OpenAI 在 2021 年發布了一個名為 codex 的模型 - 這個工具與其相關嗎？

在 2021 年，OpenAI 發布了 codex，這是一個設計用於從自然語言提示產生程式碼的 AI 系統。那個原始的 codex 模型已於 2023 年 3 月廢棄，與此 CLI 工具是分開的。

### 支援哪些模型？

我們建議使用 Codex 搭配 GPT-5，這是我們最好的程式編寫模型。預設推理級別為中等，您可以使用 `/model` 指令升級為高級以處理複雜任務。

您也可以透過使用基於 API 的驗證並使用 `--model` 標誌啟動 codex 來使用較舊的模型。

### 為什麼 `o3` 或 `o4-mini` 對我無效？

可能您的 [API 帳戶需要驗證](https://help.openai.com/en/articles/10910291-api-organization-verification) 才能開始串流回應並從 API 查看思考鏈摘要。如果您仍然遇到問題，請告訴我們！

### 如何阻止 codex 編輯我的檔案？

預設情況下，Codex 可以修改您當前工作目錄中的檔案（自動模式）。要防止編輯，請使用 CLI 標誌 `--sandbox read-only` 在唯讀模式下執行 `codex`。或者，您可以在對話中使用 `/approvals` 變更核准級別。

### 它能在 Windows 上運行嗎？

直接在 Windows 上執行 Codex 可能有效，但不受官方支援。我們建議使用 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install)。 