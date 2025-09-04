## 常見問題

### OpenAI 在 2021 年發布了一個名為 Codex 的模型 - 這個有關聯嗎？

在 2021 年，OpenAI 發布了 Codex，這是一個設計用來從自然語言提示生成程式碼的 AI 系統。該原始 Codex 模型已於 2023 年 3 月停用，且與此 CLI 工具是分離的。

### 支援哪些模型？

我們建議使用 Codex 搭配 GPT-5，這是我們最佳的編程模型。預設推理等級為中等，您可以使用 `/model` 命令升級到高等級以處理複雜任務。

您也可以透過使用基於 API 的認證並以 `--model` 旗標啟動 codex 來使用較舊的模型。

### 為什麼 `o3` 或 `o4-mini` 對我無效？

您的 [API 帳戶可能需要驗證](https://help.openai.com/en/articles/10910291-api-organization-verification) 才能開始串流回應並從 API 查看思維鏈摘要。如果您仍遇到問題，請告知我們！

### 如何阻止 Codex 編輯我的檔案？

預設情況下，Codex 可以修改您目前工作目錄中的檔案（自動模式）。要防止編輯，請使用 CLI 旗標 `--sandbox read-only` 以唯讀模式執行 `codex`。或者，您可以在對話中使用 `/approvals` 變更核准等級。

### 它在 Windows 上能運作嗎？

在 Windows 上直接執行 Codex 可能可以運作，但不受官方支援。我們建議使用 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install)。