<h1 align="center">OpenAI Codex CLI</h1>

<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install codex</code></p>

<p align="center"><strong>Codex CLI</strong> 是來自 OpenAI 的程式編寫代理，在您的電腦上本地運行。</br>如果您在尋找來自 OpenAI 的<em>基於雲端的代理</em> <strong>Codex Web</strong>，請參閱 <a href="https://chatgpt.com/codex">chatgpt.com/codex</a>。</p>

<p align="center">
  <img src="./.github/codex-cli-splash.png" alt="Codex CLI splash" width="80%" />
  </p>

---

## 快速開始

### 安裝並執行 Codex CLI

使用您偏好的套件管理器進行全域安裝。如果您使用 npm：

```shell
npm install -g @openai/codex
```

或者，如果您使用 Homebrew：

```shell
brew install codex
```

然後只需執行 `codex` 即可開始使用：

```shell
codex
```

<details>
<summary>您也可以前往 <a href="https://github.com/openai/codex/releases/latest">最新的 GitHub 發布</a> 並下載適合您平台的二進制檔案。</summary>

每個 GitHub 發布都包含多個可執行檔，但實際上，您可能需要其中之一：

- macOS
  - Apple Silicon/arm64: `codex-aarch64-apple-darwin.tar.gz`
  - x86_64 (較舊的 Mac 硬體): `codex-x86_64-apple-darwin.tar.gz`
- Linux
  - x86_64: `codex-x86_64-unknown-linux-musl.tar.gz`
  - arm64: `codex-aarch64-unknown-linux-musl.tar.gz`

每個壓縮檔都包含一個檔名內建平台名稱的單一項目（例如 `codex-x86_64-unknown-linux-musl`），因此您可能需要在解壓縮後將其重新命名為 `codex`。

</details>

### 使用您的 ChatGPT 方案運行 Codex

<p align="center">
  <img src="./.github/codex-cli-login.png" alt="Codex CLI login" width="80%" />
  </p>

執行 `codex` 並選擇 **使用 ChatGPT 登入**。我們建議登入您的 ChatGPT 帳戶，以便在您的 Plus、Pro、Team、Edu 或 Enterprise 方案中使用 Codex。[了解更多關於您的 ChatGPT 方案包含的內容](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)。

您也可以使用 API 金鑰運行 Codex，但這需要 [額外設置](./docs/authentication.md#usage-based-billing-alternative-use-an-openai-api-key)。如果您之前使用 API 金鑰進行用量計費，請參閱 [遷移步驟](./docs/authentication.md#migrating-from-usage-based-billing-api-key)。如果您在登入時遇到問題，請在 [此議題](https://github.com/openai/codex/issues/1243) 上留言。

### 模型上下文協定 (MCP)

Codex CLI 支援 [MCP 伺服器](./docs/advanced.md#model-context-protocol-mcp)。透過在您的 `~/.codex/config.toml` 中新增 `mcp_servers` 區段來啟用。

### 設定

Codex CLI 支援豐富的設定選項，偏好設定儲存在 `~/.codex/config.toml` 中。完整的設定選項請參閱 [設定](./docs/config.md)。

---

### 文件與常見問題

- [**開始使用**](./docs/getting-started.md)
  - [CLI 使用方式](./docs/getting-started.md#cli-usage)
  - [以提示作為輸入運行](./docs/getting-started.md#running-with-a-prompt-as-input)
  - [範例提示](./docs/getting-started.md#example-prompts)
  - [使用 AGENTS.md 進行記憶](./docs/getting-started.md#memory--project-docs)
  - [設定](./docs/config.md)
- [**沙盒與核准**](./docs/sandbox.md)
- [**身份驗證**](./docs/authentication.md)
  - [驗證方法](./docs/authentication.md#forcing-a-specific-auth-method-advanced)
  - [在「無頭」機器上登入](./docs/authentication.md#connecting-on-a-headless-machine)
- [**進階**](./docs/advanced.md)
  - [非互動式 / CI 模式](./docs/advanced.md#non-interactive--ci-mode)
  - [追蹤 / 詳細記錄](./docs/advanced.md#tracing--verbose-logging)
  - [模型上下文協定 (MCP)](./docs/advanced.md#model-context-protocol-mcp)
- [**零資料保留 (ZDR)**](./docs/zdr.md)
- [**貢獻**](./docs/contributing.md)
- [**安裝與建構**](./docs/install.md)
  - [系統需求](./docs/install.md#system-requirements)
  - [DotSlash](./docs/install.md#dotslash)
  - [從原始碼建構](./docs/install.md#build-from-source)
- [**常見問題**](./docs/faq.md)
- [**開源基金**](./docs/open-source-fund.md)

---

## 授權

此儲存庫依據 [Apache-2.0 授權條款](LICENSE) 授權。
