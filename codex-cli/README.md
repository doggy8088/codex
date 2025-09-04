<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Lightweight coding agent that runs in your terminal</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

> [!IMPORTANT]
> This is the documentation for the _legacy_ TypeScript implementation of the Codex CLI. It has been superseded by the _Rust_ implementation. See the [README in the root of the Codex repository](https://github.com/openai/codex/blob/main/README.md) for details.

![Codex demo GIF using: codex "explain this codebase to me"](../.github/demo.gif)

---

<details>
<summary><strong>Table of contents</strong></summary>

<!-- Begin ToC -->

- [Experimental technology disclaimer](#experimental-technology-disclaimer)
- [Quickstart](#quickstart)
- [Why Codex?](#why-codex)
- [Security model & permissions](#security-model--permissions)
  - [Platform sandboxing details](#platform-sandboxing-details)
- [System requirements](#system-requirements)
- [CLI reference](#cli-reference)
- [Memory & project docs](#memory--project-docs)
- [Non-interactive / CI mode](#non-interactive--ci-mode)
- [Tracing / verbose logging](#tracing--verbose-logging)
- [Recipes](#recipes)
- [Installation](#installation)
- [Configuration guide](#configuration-guide)
  - [Basic configuration parameters](#basic-configuration-parameters)
  - [Custom AI provider configuration](#custom-ai-provider-configuration)
  - [History configuration](#history-configuration)
  - [Configuration examples](#configuration-examples)
  - [Full configuration example](#full-configuration-example)
  - [Custom instructions](#custom-instructions)
  - [Environment variables setup](#environment-variables-setup)
- [FAQ](#faq)
- [Zero data retention (ZDR) usage](#zero-data-retention-zdr-usage)
- [Codex open source fund](#codex-open-source-fund)
- [Contributing](#contributing)
  - [Development workflow](#development-workflow)
  - [Git hooks with Husky](#git-hooks-with-husky)
  - [Debugging](#debugging)
  - [Writing high-impact code changes](#writing-high-impact-code-changes)
  - [Opening a pull request](#opening-a-pull-request)
  - [Review process](#review-process)
  - [Community values](#community-values)
  - [Getting help](#getting-help)
  - [Contributor license agreement (CLA)](#contributor-license-agreement-cla)
    - [Quick fixes](#quick-fixes)
  - [Releasing `codex`](#releasing-codex)
  - [Alternative build options](#alternative-build-options)
    - [Nix flake development](#nix-flake-development)
- [Security & responsible AI](#security--responsible-ai)
- [License](#license)

<!-- End ToC -->

</details>

---

## Experimental technology disclaimer

codex CLI is an experimental project under active 開發. 它 is not yet stable, may contain bugs, incomplete features, 或 undergo breaking changes. We're building 它 在 the open 使用 the community 和 welcome:

- Bug reports
- Feature requests
- Pull requests
- Good vibes

Help us improve 透過 filing issues 或 submitting PR (請參閱 section below 為 如何 到 contribute)!

## 快速開始

安裝 globally:

```shell
npm install -g @openai/codex
```

Next, set 您的 OpenAI API key 作為 an environment variable:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Note:** This command sets the key only for your current terminal session. You can add the `export` line to your shell's configuration file (e.g., `~/.zshrc`) but we recommend setting for the session. **Tip:** You can also place your API key into a `.env` file at the root of your project:
>
> ```env
> OPENAI_API_KEY=您的-API-key-here
> ```
>
> The CLI will automatically load variables from `.env` (via `dotenv/config`).

<details>
<summary><strong>Use <code>--provider</code> to use other models</strong></summary>

> Codex also allows you to use other providers that support the OpenAI Chat Completions API. You can set the provider in the config file or use the `--provider` flag. The possible options for `--provider` are:
>
> - openai (default)
> - openrouter
> - azure
> - gemini
> - ollama
> - mistral
> - deepseek
> - xai
> - groq
> - arceeai
> - any other provider 那個 is compatible 使用 the OpenAI API
>
> 如果 您 use a provider other than OpenAI, 您 will need 到 set the API key 為 the provider 在 the config file 或 在 the environment variable 作為:
>
> ```shell
> export <provider>_API_KEY="您的-API-key-here"
> ```
>
> 如果 您 use a provider not listed above, 您 must also set the base URL 為 the provider:
>
> ```shell
> export <provider>_BASE_URL="https://your-provider-api-base-url"
> ```

</details>
<br />

執行 interactively:

```shell
codex
```

Or, run with a prompt as input (and optionally in `Full Auto` mode):

```shell
codex "explain this codebase to me"
```

```shell
codex --approval-mode full-auto "create the fanciest todo-list app"
```

那個's 它 - codex will scaffold a file, 執行 它 inside a sandbox, 安裝 any
missing dependencies, 和 show 您 the live result. Approve the changes 和
they'll be committed 到 您的 working directory.

---

## 為什麼 codex?

codex CLI is built 為 developers who already **live 在 the terminal** 和 want
ChatGPT-level reasoning **Plus** the power 到 actually 執行 code, manipulate
檔案為 codex 提供額外的指示和指導, 和 iterate - all under version control. 在 short, 它's _chat-driven
development_ 那個 understands 和 executes 您的 repo.

- **Zero 設置** - bring 您的 OpenAI API key 和 它 just works!
- **Full auto-approval, while safe + secure** 透過 running network-disabled 和 directory-sandboxed
- **Multimodal** - pass 在 screenshots 或 diagrams 到 implement features ✨

和 它's **fully open-source** so 您 can see 和 contribute 到 如何 它 develops!

---

## Security model & permissions

codex lets 您 decide _how much autonomy_ the agent receives 和 auto-approval policy via the
`--approval-mode` flag (or the interactive onboarding prompt):

| Mode                      | 什麼 the agent may do without asking                                                                | Still requires approval                                                                         |
| ------------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Suggest** <br>(default) | <li>Read any file in the repo                                                                       | <li>**All** file writes/patches<li> **Any** arbitrary shell commands (aside from reading files) |
| **Auto Edit**             | <li>Read **and** apply-patch writes to files                                                        | <li>**All** shell commands                                                                      |
| **Full Auto**             | <li>Read/write files <li> Execute shell commands (network disabled, writes limited to your workdir) | -                                                                                               |

在 **Full Auto** every 指令 is 執行 **network-disabled** 和 confined 到 the
current working directory (Plus temporary 檔案為 codex 提供額外的指示和指導) 為 defense-在-depth. codex
will also show a warning/confirmation 如果 您 start 在 **auto-edit** 或
**full-auto** while the directory is _not_ tracked 透過 git, so 您 always have a
safety net.

Coming soon: 您'll be able 到 whitelist specific commands 到 auto-execute 使用
the network enabled, once we're confident 在 additional safeguards.

### Platform sandboxing details

The hardening mechanism codex uses depends 在 您的 OS:

- **macOS 12+** - commands are wrapped with **Apple Seatbelt** (`sandbox-exec`).

  - Everything is placed 在 a read-only jail except 為 a small set 的
    writable roots (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - Outbound network is _fully blocked_ 透過 default - even 如果 a child process
    tries to `curl` somewhere it will fail.

- **Linux** - there is no sandboxing 透過 default.
  We recommend using Docker 為 sandboxing, 哪裡 codex launches itself inside a **minimal
  container image** and mounts your repo _read/write_ at the same path. A
  custom `iptables`/`ipset` firewall script denies all egress except the
  OpenAI API. 這個 gives 您 deterministic, reproducible runs without needing
  root on the host. You can use the [`run_in_container.sh`](../codex-cli/scripts/run_in_container.sh) script to set up the sandbox.

---

## 系統需求

| Requirement                 | Details                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Operating systems           | macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 **via WSL2** |
| Node.js                     | **22 或 newer** (LTS recommended)                               |
| git (optional, recommended) | 2.23+ 為 built-在 PR helpers                                   |
| RAM                         | 4-GB minimum (8-GB recommended)                                 |

> Never run `sudo npm install -g`; fix npm permissions instead.

---

## CLI reference

| 指令                              | 目的                             | 範例                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | Interactive REPL                    | `codex`                              |
| `codex "..."`                        | Initial prompt for interactive REPL | `codex "fix lint errors"`            |
| `codex -q "..."`                     | Non-interactive "quiet mode"        | `codex -q --json "explain utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Print shell completion script       | `codex completion bash`              |

Key flags: `--model/-m`, `--approval-mode/-a`, `--quiet/-q`, and `--notify`.

---

## Memory & project docs

You can give Codex extra instructions and guidance using `AGENTS.md` files. Codex looks for `AGENTS.md` files in the following places, and merges them top-down:

1. `~/.codex/AGENTS.md` - personal global guidance
2. `AGENTS.md` at repo root - shared project notes
3. `AGENTS.md` in the current working directory - sub-folder/feature specifics

Disable loading of these files with `--no-project-doc` or the environment variable `CODEX_DISABLE_PROJECT_DOC=1`.

---

## 非互動式 / CI 模式

執行 codex head-less 在 pipelines. 範例 GitHub Action step:

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "update CHANGELOG for next release"
```

Set `CODEX_QUIET_MODE=1` to silence interactive UI noise.

## 追蹤 / 詳細記錄

Setting the environment variable `DEBUG=true` prints full API request and response details:

```shell
DEBUG=true codex
```

---

## Recipes

Below are a few bite-size examples you can copy-paste. Replace the text in quotes with your own task. See the [prompting guide](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) for more tips and usage patterns.

| ✨  | 您輸入的內容                                                                   | 會發生什麼                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex rewrites the class component, runs `npm test`, and shows the diff.   |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | Infers your ORM, creates migration files, and runs them in a sandboxed DB. |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | Generates tests, executes them, and iterates until they pass.              |
| 4   | `codex "Bulk-rename *.jpeg -> *.jpg with git mv"`                               | Safely renames files and updates imports/usages.                           |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | Outputs a step-by-step human explanation.                                  |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | Suggests impactful PRs in the current codebase.                            |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | Finds and explains security bugs.                                          |

---

## 安裝

<details open>
<summary><strong>From npm (Recommended)</strong></summary>

```bash
npm install -g @openai/codex
# or
yarn global add @openai/codex
# or
bun install -g @openai/codex
# or
pnpm add -g @openai/codex
```

</details>

<details>
<summary><strong>Build from source</strong></summary>

```bash
# Clone the repository and navigate to the CLI package
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Enable corepack
corepack enable

# Install dependencies and build
pnpm install
pnpm build

# Linux-only: download prebuilt sandboxing binaries (requires gh and zstd).
./scripts/install_native_deps.sh

# Get the usage and the options
node ./dist/cli.js --help

# Run the locally-built CLI directly
node ./dist/cli.js

# Or link the command globally for convenience
pnpm link
```

</details>

---

## 設定 guide

Codex configuration files can be placed in the `~/.codex/` directory, supporting both YAML and JSON formats.

### Basic 設定 parameters

| Parameter           | Type    | Default    | Description                      | Available Options                                                                              |
| ------------------- | ------- | ---------- | -------------------------------- | ---------------------------------------------------------------------------------------------- |
| `model`             | string  | `o4-mini`  | AI model to use                  | Any model name supporting OpenAI API                                                           |
| `approvalMode`      | string  | `suggest`  | AI assistant's permission mode   | `suggest` (suggestions only)<br>`auto-edit` (automatic edits)<br>`full-auto` (fully automatic) |
| `fullAutoErrorMode` | string  | `ask-user` | Error handling in full-auto mode | `ask-user` (prompt for user input)<br>`ignore-and-continue` (ignore and proceed)               |
| `notify`            | boolean | `true`     | Enable desktop notifications     | `true`/`false`                                                                                 |

### Custom AI provider 設定

In the `providers` object, you can configure multiple AI service providers. Each provider requires the following parameters:

| Parameter | Type   | Description                             | 範例                       |
| --------- | ------ | --------------------------------------- | ----------------------------- |
| `name`    | string | Display name of the provider            | `"OpenAI"`                    |
| `baseURL` | string | API service URL                         | `"https://api.openai.com/v1"` |
| `envKey`  | string | Environment variable name (for API key) | `"OPENAI_API_KEY"`            |

### History 設定

In the `history` object, you can configure conversation history settings:

| Parameter           | Type    | Description                                            | 範例 Value |
| ------------------- | ------- | ------------------------------------------------------ | ------------- |
| `maxSize`           | number  | Maximum number of history entries to save              | `1000`        |
| `saveHistory`       | boolean | Whether to save history                                | `true`        |
| `sensitivePatterns` | array   | Patterns of sensitive information to filter in history | `[]`          |

### 設定 範例

1. YAML format (save as `~/.codex/config.yaml`):

```yaml
model: o4-mini
approvalMode: suggest
fullAutoErrorMode: ask-user
notify: true
```

2. JSON format (save as `~/.codex/config.json`):

```json
{
  "model": "o4-mini",
  "approvalMode": "suggest",
  "fullAutoErrorMode": "ask-user",
  "notify": true
}
```

### Full 設定 範例

Below is a comprehensive example of `config.json` with multiple custom providers:

```json
{
  "model": "o4-mini",
  "provider": "openai",
  "providers": {
    "openai": {
      "name": "OpenAI",
      "baseURL": "https://api.openai.com/v1",
      "envKey": "OPENAI_API_KEY"
    },
    "azure": {
      "name": "AzureOpenAI",
      "baseURL": "https://YOUR_PROJECT_NAME.openai.azure.com/openai",
      "envKey": "AZURE_OPENAI_API_KEY"
    },
    "openrouter": {
      "name": "OpenRouter",
      "baseURL": "https://openrouter.ai/api/v1",
      "envKey": "OPENROUTER_API_KEY"
    },
    "gemini": {
      "name": "Gemini",
      "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai",
      "envKey": "GEMINI_API_KEY"
    },
    "ollama": {
      "name": "Ollama",
      "baseURL": "http://localhost:11434/v1",
      "envKey": "OLLAMA_API_KEY"
    },
    "mistral": {
      "name": "Mistral",
      "baseURL": "https://api.mistral.ai/v1",
      "envKey": "MISTRAL_API_KEY"
    },
    "deepseek": {
      "name": "DeepSeek",
      "baseURL": "https://api.deepseek.com",
      "envKey": "DEEPSEEK_API_KEY"
    },
    "xai": {
      "name": "xAI",
      "baseURL": "https://api.x.ai/v1",
      "envKey": "XAI_API_KEY"
    },
    "groq": {
      "name": "Groq",
      "baseURL": "https://api.groq.com/openai/v1",
      "envKey": "GROQ_API_KEY"
    },
    "arceeai": {
      "name": "ArceeAI",
      "baseURL": "https://conductor.arcee.ai/v1",
      "envKey": "ARCEEAI_API_KEY"
    }
  },
  "history": {
    "maxSize": 1000,
    "saveHistory": true,
    "sensitivePatterns": []
  }
}
```

### Custom instructions

You can create a `~/.codex/AGENTS.md` file to define custom guidance for the agent:

```markdown
- Always respond with emojis
- Only use git commands when explicitly requested
```

### Environment variables 設置

為 each AI provider, 您 need 到 set the corresponding API key 在 您的 environment variables. 為 範例:

```bash
# OpenAI
export OPENAI_API_KEY="your-api-key-here"

# Azure OpenAI
export AZURE_OPENAI_API_KEY="your-azure-api-key-here"
export AZURE_OPENAI_API_VERSION="2025-04-01-preview" (Optional)

# OpenRouter
export OPENROUTER_API_KEY="your-openrouter-key-here"

# Similarly for other providers
```

---

## 常見問題

<details>
<summary>OpenAI released a model called Codex in 2021 - is this related?</summary>

在 2021, OpenAI released codex, an AI system designed 到 generate code 來自 natural language prompts. 那個 original codex model was deprecated 作為 的 March 2023 和 is separate 來自 the CLI tool.

</details>

<details>
<summary>Which models are supported?</summary>

Any model available with [Responses API](https://platform.openai.com/docs/api-reference/responses). The default is `o4-mini`, but pass `--model gpt-4.1` or set `model: gpt-4.1` in your config file to override.

</details>
<details>
<summary>Why does <code>o3</code> or <code>o4-mini</code> not work for me?</summary>

It's possible that your [API account needs to be verified](https://help.openai.com/en/articles/10910291-api-organization-verification) in order to start streaming responses and seeing chain of thought summaries from the API. If you're still running into issues, please let us know!

</details>

<details>
<summary>How do I stop Codex from editing my files?</summary>

codex runs model-generated commands 在 a sandbox. 如果 a proposed 指令 或 file change doesn't look right, 您 can simply type **n** 到 deny the 指令 或 give the model feedback.

</details>
<details>
<summary>Does it work on Windows?</summary>

Not directly. It requires [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) - Codex has been tested on macOS and Linux with Node 22.

</details>

---

## Zero data retention (ZDR) usage

Codex CLI **does** support OpenAI organizations with [Zero Data Retention (ZDR)](https://platform.openai.com/docs/guides/your-data#zero-data-retention) enabled. If your OpenAI organization has Zero Data Retention enabled and you still encounter errors such as:

```
OpenAI rejected the request. Error details: Status: 400, Code: unsupported_parameter, Type: invalid_request_error, Message: 400 Previous response cannot be used for this organization due to Zero Data Retention.
```

You may need to upgrade to a more recent version with: `npm i -g @openai/codex@latest`

---

## codex 開源基金

We're excited 到 launch a **$1 million initiative** supporting open source projects 那個 use codex CLI 和 other OpenAI models.

- Grants are awarded up 到 **$25,000** API credits.
- Applications are reviewed **在 a rolling basis**.

**Interested? [Apply here](https://openai.com/form/codex-open-source-fund/).**

---

## 貢獻

這個 project is under active 開發 和 the code will likely change pretty significantly. We'll 更新 這個 message once 那個's complete!

More broadly we welcome contributions - whether 您 are opening 您的 very first pull request 或 您're a seasoned maintainer. 在 the same time we care about reliability 和 long-term maintainability, so the bar 為 merging code is intentionally **high**. The guidelines below spell out 什麼 "high-quality" means 在 practice 和 should make the whole process transparent 和 friendly.

### 開發工作流程

- Create a _topic branch_ from `main` - e.g. `feat/interactive-prompt`.
- Keep 您的 changes focused. Multiple unrelated fixes should be opened 作為 separate PR.
- Use `pnpm test:watch` during development for super-fast feedback.
- We use **Vitest** 為 unit tests, **ESLint** + **Prettier** 為 style, 和 **TypeScript** 為 type-checking.
- Before pushing, 執行 the full 測試/type/lint suite:

### git Hooks 使用 Husky

This project uses [Husky](https://typicode.github.io/husky/) to enforce code quality checks:

- **Pre-commit hook**: Automatically runs lint-staged 到 format 和 lint 檔案為 codex 提供額外的指示和指導 before committing
- **Pre-push hook**: Runs tests 和 type checking before pushing 到 the remote

These hooks help maintain code quality and prevent pushing code with failing tests. For more details, see [HUSKY.md](./HUSKY.md).

```bash
pnpm test && pnpm run lint && pnpm run typecheck
```

- 如果 您 have **not** yet signed the Contributor 授權 Agreement (CLA), add a PR comment containing the exact text

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  The CLA-Assistant bot will turn the PR status green once all authors have signed.

```bash
# Watch mode (tests rerun on change)
pnpm test:watch

# Type-check without emitting files
pnpm typecheck

# Automatically fix lint + prettier issues
pnpm lint:fix
pnpm format:fix
```

### Debugging

To debug the CLI with a visual debugger, do the following in the `codex-cli` folder:

- Run `pnpm run build` to build the CLI, which will generate `cli.js.map` alongside `cli.js` in the `dist` folder.
- Run the CLI with `node --inspect-brk ./dist/cli.js` The program then waits until a debugger is attached before proceeding. Options:
  - In VS Code, choose **Debug: Attach to Node Process** from the command palette and choose the option in the dropdown with debug port `9229` (likely the first option)
  - Go to <chrome://inspect> in Chrome and find **localhost:9229** and click **trace**

### 撰寫高影響力的程式碼變更

1. **Start 使用 an issue.** Open a new one 或 comment 在 an existing discussion so we can agree 在 the solution before code is written.
2. **Add 或 更新 tests.** Every new feature 或 bug-fix should come 使用 測試 coverage 那個 fails before 您的 change 和 passes afterwards. 100% coverage is not required, but aim 為 meaningful assertions.
3. **Document behaviour.** If your change affects user-facing behaviour, update the README, inline help (`codex --help`), or relevant example projects.
4. **Keep commits atomic.** Each commit should compile 和 the tests should pass. 這個 makes reviews 和 potential rollbacks easier.

### 開啟拉取請求

- Fill 在 the PR template (或 include similar information) - **什麼? 為什麼? 如何?**
- Run **all** checks locally (`npm test && npm run lint && npm run typecheck`). CI failures that could have been caught locally slow down the process.
- Make sure your branch is up-to-date with `main` and that you have resolved merge conflicts.
- Mark the PR 作為 **Ready 為 review** only 當 您 believe 它 is 在 a merge-able state.

### 審查流程

1. One maintainer will be assigned 作為 a primary reviewer.
2. We may ask 為 changes - please do not take 這個 personally. We value the work, we just also value consistency 和 long-term maintainability.
3. 當 there is consensus 那個 the PR meets the bar, a maintainer will squash-和-merge.

### Community values

- **Be kind and inclusive.** Treat others with respect; we follow the [Contributor Covenant](https://www.contributor-covenant.org/).
- **Assume good intent.** Written communication is hard - err 在 the side 的 generosity.
- **Teach & learn.** 如果 您 spot something confusing, open an issue 或 PR 使用 improvements.

### Getting help

如果 您 執行 into problems setting up the project, would like feedback 在 an idea, 或 just want 到 say _hi_ - please open a Discussion 或 jump into the relevant issue. We are happy 到 help.

Together we can make codex CLI an incredible tool. **Happy hacking!** :rocket:

### Contributor 授權 agreement (CLA)

All contributors **must** accept the CLA. The process is lightweight:

1. Open 您的 pull request.
2. Paste the following comment (or reply `recheck` if you've signed before):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. The CLA-Assistant bot records 您的 signature 在 the repo 和 marks the status check 作為 passed.

No special git commands, email attachments, 或 commit footers required.

#### Quick fixes

| Scenario          | 指令                                          |
| ----------------- | ------------------------------------------------ |
| Amend last commit | `git commit --amend -s --no-edit && git push -f` |

The **DCO check** blocks merges until every commit 在 the PR carries the footer (使用 squash 這個 is just the one).

### Releasing `codex`

到 publish a new version 的 the CLI 您 first need 到 stage the npm package. A
helper script in `codex-cli/scripts/` does all the heavy lifting. Inside the
`codex-cli` folder run:

```bash
# Classic, JS implementation that includes small, native binaries for Linux sandboxing.
pnpm stage-release

# Optionally specify the temp directory to reuse between runs.
RELEASE_DIR=$(mktemp -d)
pnpm stage-release --tmp "$RELEASE_DIR"

# "Fat" package that additionally bundles the native Rust CLI binaries for
# Linux. End-users can then opt-in at runtime by setting CODEX_RUST=1.
pnpm stage-release --native
```

Go 到 the folder 哪裡 the release is staged 和 verify 那個 它 works 作為 intended. 如果 so, 執行 the following 來自 the temp folder:

```
cd "$RELEASE_DIR"
npm publish
```

### Alternative 建構 options

#### Nix flake 開發

Prerequisite: Nix >= 2.4 with flakes enabled (`experimental-features = nix-command flakes` in `~/.config/nix/nix.conf`).

Enter a Nix 開發 shell:

```bash
# Use either one of the commands according to which implementation you want to work with
nix develop .#codex-cli # For entering codex-cli specific shell
nix develop .#codex-rs # For entering codex-rs specific shell
```

This shell includes Node.js, installs dependencies, builds the CLI, and provides a `codex` command alias.

建構 和 執行 the CLI directly:

```bash
# Use either one of the commands according to which implementation you want to work with
nix build .#codex-cli # For building codex-cli
nix build .#codex-rs # For building codex-rs
./result/bin/codex --help
```

執行 the CLI via the flake app:

```bash
# Use either one of the commands according to which implementation you want to work with
nix run .#codex-cli # For running codex-cli
nix run .#codex-rs # For running codex-rs
```

Use direnv 使用 flakes

If you have direnv installed, you can use the following `.envrc` to automatically enter the Nix shell when you `cd` into the project directory:

```bash
cd codex-rs
echo "use flake ../flake.nix#codex-cli" >> .envrc && direnv allow
cd codex-cli
echo "use flake ../flake.nix#codex-rs" >> .envrc && direnv allow
```

---

## Security & responsible AI

Have you discovered a vulnerability or have concerns about model output? Please e-mail **security@openai.com** and we will respond promptly.

---

## 授權

This repository is licensed under the [Apache-2.0 License](LICENSE).
