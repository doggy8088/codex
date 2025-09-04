<h1 align="center">OpenAI Codex CLI</h1>

<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install codex</code></p>

<p align="center"><strong>Codex CLI</strong> 是 OpenAI 的一個編碼代理程式，它會在您的電腦上本機執行。</br>如果您正在尋找 OpenAI 的<em>雲端代理程式</em> <strong>Codex Web</strong>，請參閱 <a href="https://chatgpt.com/codex">chatgpt.com/codex</a>。</p>

<p align="center">
  <img src="./.github/codex-cli-splash.png" alt="Codex CLI 啟動畫面" width="80%" />
  </p>

---

## 快速入門

### 安裝與執行 Codex CLI

使用您偏好的套件管理器進行全域安裝。如果您使用 npm：

```shell
npm install -g @openai/codex
```

或者，如果您使用 Homebrew：

```shell
brew install codex
```

然後只需執行 `codex` 即可開始：

```shell
codex
```

<details>
<summary>您也可以前往<a href="https://github.com/openai/codex/releases/latest">最新的 GitHub Release</a>，並為您的平台下載對應的二進位檔案。</summary>

每個 GitHub Release 都包含許多可執行檔，但在實務上，您可能只需要其中一個：

- macOS
  - Apple Silicon/arm64：`codex-aarch64-apple-darwin.tar.gz`
  - x86_64（較舊的 Mac 硬體）：`codex-x86_64-apple-darwin.tar.gz`
- Linux
  - x86_64：`codex-x86_64-unknown-linux-musl.tar.gz`
  - arm64：`codex-aarch64-unknown-linux-musl.tar.gz`

每個封存檔都包含一個單獨的項目，其名稱中包含了平台資訊（例如 `codex-x86_64-unknown-linux-musl`），所以您可能會想在解壓縮後將其重新命名為 `codex`。

</details>

### 使用 Codex 搭配您的 ChatGPT 方案

<p align="center">
  <img src="./.github/codex-cli-login.png" alt="Codex CLI 登入" width="80%" />
  </p>

執行 `codex` 並選擇 **Sign in with ChatGPT**。我們建議您登入 ChatGPT 帳戶，以將 Codex 作為您 Plus、Pro、Team、Edu 或 Enterprise 方案的一部分來使用。[進一步了解您的 ChatGPT 方案包含哪些內容](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)。

您也可以使用 API 金鑰來使用 Codex，但這需要[額外設定](./docs/authentication.md#usage-based-billing-alternative-use-an-openai-api-key)。如果您先前曾使用 API 金鑰進行用量計費，請參閱[遷移步驟](./docs/authentication.md#migrating-from-usage-based-billing-api-key)。如果您在登入時遇到問題，請在[此議題](https://github.com/openai/codex/issues/1243)上留言。

### 模型上下文協定 (MCP)

Codex CLI 支援 [MCP 伺服器](./docs/advanced.md#model-context-protocol-mcp)。若要啟用，請在您的 `~/.codex/config.toml` 中新增一個 `mcp_servers` 區段。


### 組態設定

Codex CLI 支援豐富的組態設定選項，偏好設定儲存在 `~/.codex/config.toml` 中。有關完整的組態設定選項，請參閱[組態設定](./docs/config.md)。

---

### 文件與常見問答

- [**入門指南**](./docs/getting-started.md)
  - [CLI 用法](./docs/getting-started.md#cli-usage)
  - [使用提示作為輸入執行](./docs/getting-started.md#running-with-a-prompt-as-input)
  - [提示範例](./docs/getting-started.md#example-prompts)
  - [使用 AGENTS.md 的記憶體](./docs/getting-started.md#memory--project-docs)
  - [組態設定](./docs/config.md)
- [**沙箱與核准**](./docs/sandbox.md)
- [**驗證**](./docs/authentication.md)
  - [驗證方法](./docs/authentication.md#forcing-a-specific-auth-method-advanced)
  - [在「無頭」機器上登入](./docs/authentication.md#connecting-on-a-headless-machine)
- [**進階**](./docs/advanced.md)
  - [非互動式 / CI 模式](./docs/advanced.md#non-interactive--ci-mode)
  - [追蹤 / 詳細日誌記錄](./docs/advanced.md#tracing--verbose-logging)
  - [模型上下文協定 (MCP)](./docs/advanced.md#model-context-protocol-mcp)
- [**零資料保留 (ZDR)**](./docs/zdr.md)
- [**貢獻**](./docs/contributing.md)
- [**安裝與建構**](./docs/install.md)
  - [系統需求](./docs/install.md#system-requirements)
  - [DotSlash](./docs/install.md#dotslash)
  - [從原始碼建構](./docs/install.md#build-from-source)
- [**常見問答**](./docs/faq.md)
- [**開源基金**](./docs/open-source-fund.md)

---

## 授權條款

本儲存庫採用 [Apache-2.0 授權條款](LICENSE) 授權。

