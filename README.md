<h1 align="center">OpenAI Codex CLI</h1>

<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install codex</code></p>

<p align="center"><strong>Codex CLI</strong> 是由 OpenAI 推出的程式設計代理，能在您的電腦本機執行。</br>如果您在尋找 OpenAI 的雲端版代理 <strong>Codex Web</strong>，請參閱 <a href="https://chatgpt.com/codex">chatgpt.com/codex</a>。</p>

<p align="center">
  <img src="./.github/codex-cli-splash.png" alt="Codex CLI 首圖" width="80%" />
  </p>

---

## 快速開始

### 安裝與執行 Codex CLI

使用您偏好的套件管理器進行全域安裝。如果您使用 npm：

```shell
npm install -g @openai/codex
```

或者，如果您使用 Homebrew：

```shell
brew install codex
```

接著執行 `codex` 以開始使用：

```shell
codex
```

<details>
<summary>您也可以前往 <a href="https://github.com/openai/codex/releases/latest">最新 GitHub Release</a>，下載適用於您平台的二進位檔。</summary>

每個 GitHub Release 都包含許多可執行檔，但實務上您大多只需要下列其中之一：

- macOS
  - Apple Silicon/arm64：`codex-aarch64-apple-darwin.tar.gz`
  - x86_64 (較舊的 Mac 硬體)：`codex-x86_64-apple-darwin.tar.gz`
- Linux
  - x86_64：`codex-x86_64-unknown-linux-musl.tar.gz`
  - arm64：`codex-aarch64-unknown-linux-musl.tar.gz`

每個壓縮檔都只包含一個項目，且其名稱含有平台資訊 (例如，`codex-x86_64-unknown-linux-musl`)，因此解壓縮後您可能會想將其重新命名為 `codex`。

</details>

### 搭配您的 ChatGPT 方案使用 Codex

<p align="center">
  <img src="./.github/codex-cli-login.png" alt="Codex CLI 登入畫面" width="80%" />
  </p>

執行 `codex` 並選擇 **Sign in with ChatGPT**。建議您使用 ChatGPT 帳戶登入，將 Codex 納入您的 Plus、Pro、Team、Edu 或 Enterprise 方案。[深入了解您的 ChatGPT 方案包含哪些內容](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)。

您也可以使用 API 金鑰來搭配 Codex，但這需要[額外設定](./docs/authentication.md#usage-based-billing-alternative-use-an-openai-api-key)。如果您先前以 API 金鑰進行用量計費，請參考[遷移步驟](./docs/authentication.md#migrating-from-usage-based-billing-api-key)。若您在登入上遇到問題，請在[此議題](https://github.com/openai/codex/issues/1243)留言。

### 模型上下文協定 (MCP)

Codex CLI 支援 [MCP 伺服器](./docs/advanced.md#model-context-protocol-mcp)。可在 `~/.codex/config.toml` 中新增 `mcp_servers` 區段以啟用。

### 設定

Codex CLI 提供豐富的設定選項，偏好設定儲存在 `~/.codex/config.toml`。完整設定選項請參閱[設定](./docs/config.md)。

---

### 文件與常見問題

- [**快速上手**](./docs/getting-started.md)
  - [CLI 用法](./docs/getting-started.md#cli-usage)
  - [以提示作為輸入執行](./docs/getting-started.md#running-with-a-prompt-as-input)
  - [提示範例](./docs/getting-started.md#example-prompts)
  - [搭配 AGENTS.md 的記憶功能](./docs/getting-started.md#memory--project-docs)
  - [設定](./docs/config.md)
- [**沙箱與核准**](./docs/sandbox.md)
- [**身分驗證**](./docs/authentication.md)
  - [驗證方式](./docs/authentication.md#forcing-a-specific-auth-method-advanced)
  - [在「Headless」機器上登入](./docs/authentication.md#connecting-on-a-headless-machine)
- [**進階**](./docs/advanced.md)
  - [非互動/CI 模式](./docs/advanced.md#non-interactive--ci-mode)
  - [追蹤/詳細記錄](./docs/advanced.md#tracing--verbose-logging)
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

此儲存庫採用 [Apache-2.0 授權](LICENSE)。
