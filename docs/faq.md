## 常見問題

### 2021 年 OpenAI 釋出的 Codex 模型與此相同嗎？

在 2021 年，OpenAI 推出 Codex——一個可由自然語言提示產生程式碼的 AI 系統。該舊版 Codex 模型已於 2023 年 3 月淘汰，與本 CLI 工具無關。

### 支援哪些模型？

我們建議搭配 GPT‑5 使用 Codex，這是我們最佳的程式設計模型。預設的 reasoning 等級為 medium，您可在複雜任務中透過 `/model` 指令升級為 high。

您也可以使用 API 驗證，並以 `--model` 旗標啟動 codex 以使用較舊的模型。

### 為什麼我無法使用 `o3` 或 `o4-mini`？

可能是因為您需要先完成［<a href="https://help.openai.com/en/articles/10910291-api-organization-verification">API 帳戶驗證</a>］，才能從 API 取得串流回應與推理摘要。若仍有問題，請告訴我們！

### 如何阻止 Codex 編輯我的檔案？

預設情況下，Codex 可修改您目前工作目錄的檔案（Auto 模式）。若要避免被編輯，請以 `--sandbox read-only` 旗標在唯讀模式下執行 `codex`。或者，您也可在對話過程中透過 `/approvals` 調整核准層級。

### 能在 Windows 上使用嗎？

在 Windows 上直接執行 Codex 可能可行，但尚未正式支援。我們建議使用［<a href="https://learn.microsoft.com/en-us/windows/wsl/install">Windows Subsystem for Linux（WSL2）</a>］。
