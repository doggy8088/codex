<h1 align="center">OpenAI Codex CLI</h1>

<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install codex</code></p>

<p align="center"><strong>Codex CLI</strong> 是由 OpenAI 推出的本機端程式設計代理。</br>如果您在尋找 OpenAI 的<em>雲端版代理</em>（Codex Web），請前往 <a href="https://chatgpt.com/codex">chatgpt.com/codex</a>。</p>

<p align="center">
  <img src="./.github/codex-cli-splash.png" alt="Codex CLI 啟動畫面" width="80%" />
  </p>

---

## 快速開始

### 安裝並執行 Codex CLI

使用您慣用的套件管理器全域安裝。若使用 npm：

```shell
npm install -g @openai/codex
```

或若使用 Homebrew：

```shell
brew install codex
```

之後直接執行 `codex` 開始使用：

```shell
codex
```

<details>
<summary>您也可以前往 <a href="https://github.com/openai/codex/releases/latest">最新的 GitHub Release</a>，下載適合您作業系統的執行檔。</summary>

每個 GitHub Release 都包含多個可執行檔，不過實際上您大多只會需要以下其中一個：

- macOS
  - Apple Silicon/arm64：`codex-aarch64-apple-darwin.tar.gz`
  - x86_64（較舊的 Mac 硬體）：`codex-x86_64-apple-darwin.tar.gz`
- Linux
  - x86_64：`codex-x86_64-unknown-linux-musl.tar.gz`
  - arm64：`codex-aarch64-unknown-linux-musl.tar.gz`

每個壓縮檔都只包含一個以平台為名稱的檔案（例如 `codex-x86_64-unknown-linux-musl`），解壓縮後您通常會想將它重新命名為 `codex`。

</details>

### 搭配您的 ChatGPT 方案使用 Codex

<p align="center">
  <img src="./.github/codex-cli-login.png" alt="Codex CLI 登入畫面" width="80%" />
  </p>

執行 `codex`，然後選擇「**Sign in with ChatGPT**」。我們建議您登入 ChatGPT 帳號，將 Codex 納入您的 Plus、Pro、Team、Edu 或 Enterprise 方案中使用。請參閱［<a href="https://help.openai.com/en/articles/11369540-codex-in-chatgpt">瞭解 ChatGPT 方案包含哪些項目</a>］。

您也可以使用 API 金鑰來搭配 Codex，但需要進行［<a href="./docs/authentication.md#usage-based-billing-alternative-use-an-openai-api-key">額外設定</a>］。如果您先前以 API 金鑰進行依使用量計費，請參考［<a href="./docs/authentication.md#migrating-from-usage-based-billing-api-key">移轉步驟</a>］。若登入時遇到問題，歡迎在［<a href="https://github.com/openai/codex/issues/1243">此議題</a>］留言回報。

### Model Context Protocol（MCP）

Codex CLI 支援［<a href="./docs/advanced.md#model-context-protocol-mcp">MCP 伺服器</a>］。請在您的 `~/.codex/config.toml` 中加入 `mcp_servers` 區段以啟用。


### 設定

Codex CLI 提供豐富的設定選項，偏好設定會儲存在 `~/.codex/config.toml`。完整設定選項請參閱［<a href="./docs/config.md">設定</a>］。

---

### 文件與常見問題

- ［<a href="./docs/getting-started.md"><strong>快速上手</strong></a>］
  - ［<a href="./docs/getting-started.md#cli-usage">CLI 用法</a>］
  - ［<a href="./docs/getting-started.md#running-with-a-prompt-as-input">以提示字串作為輸入執行</a>］
  - ［<a href="./docs/getting-started.md#example-prompts">提示字串範例</a>］
  - ［<a href="./docs/getting-started.md#memory--project-docs">使用 AGENTS.md 的記憶體</a>］
  - ［<a href="./docs/config.md">設定</a>］
- ［<a href="./docs/sandbox.md"><strong>沙盒與核准</strong></a>］
- ［<a href="./docs/authentication.md"><strong>驗證</strong></a>］
  - ［<a href="./docs/authentication.md#forcing-a-specific-auth-method-advanced">驗證方式</a>］
  - ［<a href="./docs/authentication.md#connecting-on-a-headless-machine">在「無頭」機器上登入</a>］
- ［<a href="./docs/advanced.md"><strong>進階</strong></a>］
  - ［<a href="./docs/advanced.md#non-interactive--ci-mode">非互動/CI 模式</a>］
  - ［<a href="./docs/advanced.md#tracing--verbose-logging">追蹤/詳細記錄</a>］
  - ［<a href="./docs/advanced.md#model-context-protocol-mcp">Model Context Protocol（MCP）</a>］
- ［<a href="./docs/zdr.md"><strong>零資料保留（ZDR）</strong></a>］
- ［<a href="./docs/contributing.md"><strong>參與貢獻</strong></a>］
- ［<a href="./docs/install.md"><strong>安裝與建構</strong></a>］
  - ［<a href="./docs/install.md#system-requirements">系統需求</a>］
  - ［<a href="./docs/install.md#dotslash">DotSlash</a>］
  - ［<a href="./docs/install.md#build-from-source">從原始碼建構</a>］
- ［<a href="./docs/faq.md"><strong>常見問題</strong></a>］
- ［<a href="./docs/open-source-fund.md"><strong>開源基金</strong></a>］

---

## 授權

此儲存庫採用［<a href="LICENSE">Apache-2.0 授權條款</a>］。
