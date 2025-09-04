# codex CLI (Rust Implementation)

We provide codex CLI 作為 a standalone, native executable 到 ensure a zero-dependency 安裝.

## Installing codex

Today, the easiest way to install Codex is via `npm`, though we plan to publish Codex to other package managers soon.

```shell
npm i -g @openai/codex@native
codex
```

You can also download a platform-specific release directly from our [GitHub Releases](https://github.com/openai/codex/releases).

## 什麼's new 在 the Rust CLI

While we are [working to close the gap between the TypeScript and Rust implementations of Codex CLI](https://github.com/openai/codex/issues/1262), note that the Rust CLI has a number of features that the TypeScript CLI does not!

### Config

Codex supports a rich set of configuration options. Note that the Rust CLI uses `config.toml` instead of `config.json`. See [`docs/config.md`](../docs/config.md) for details.

### Model Context Protocol Support

Codex CLI functions as an MCP client that can connect to MCP servers on startup. See the [`mcp_servers`](../docs/config.md#mcp_servers) section in the configuration documentation for details.

It is still experimental, but you can also launch Codex as an MCP _server_ by running `codex mcp`. Use the [`@modelcontextprotocol/inspector`](https://github.com/modelcontextprotocol/inspector) to try it out:

```shell
npx @modelcontextprotocol/inspector codex mcp
```

### Notifications

You can enable notifications by configuring a script that is run whenever the agent finishes a turn. The [notify documentation](../docs/config.md#notify) includes a detailed example that explains how to get desktop notifications via [terminal-notifier](https://github.com/julienXX/terminal-notifier) on macOS.

### `codex exec` to run Codex programmatially/non-interactively

To run Codex non-interactively, run `codex exec PROMPT` (you can also pass the prompt via `stdin`) and Codex will work on your task until it decides that it is done and exits. Output is printed to the terminal directly. You can set the `RUST_LOG` environment variable to see more about what's going on.

### Use `@` for file search

Typing `@` triggers a fuzzy-filename search over the workspace root. Use up/down to select among the results and Tab or Enter to replace the `@` with the selected path. You can use Esc to cancel the search.

### Esc–Esc 編輯上一則訊息

當 the chat composer is empty, press Esc 到 prime “backtrack” mode. 再次按 Esc 開啟突出顯示最後一則使用者訊息的對話預覽; 重複按 Esc 可回到更舊的使用者訊息. 按 Enter 確認，codex 將從該點分叉對話, 相應地修剪可見對話記錄, 並用選取的使用者訊息預填編輯器，以便您可以編輯並重新提交.

In the transcript preview, the footer shows an `Esc edit prev` hint while editing is active.

### `--cd`/`-C` flag

Sometimes it is not convenient to `cd` to the directory you want Codex to use as the "working root" before running Codex. Fortunately, `codex` supports a `--cd` option so you can specify whatever folder you want. You can confirm that Codex is honoring `--cd` by double-checking the **workdir** it reports in the TUI at the start of a new session.

### Shell 自動完成

透過以下方式產生 shell 自動完成腳本:

```shell
codex completion bash
codex completion zsh
codex completion fish
```

### Experimenting 使用 the codex Sandbox

到 測試 到 see 會發生什麼 當 a 指令 is 執行 under the sandbox provided 透過 codex, we provide the following subcommands 在 codex CLI:

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### Selecting a sandbox policy via `--sandbox`

The Rust CLI exposes a dedicated `--sandbox` (`-s`) flag that lets you pick the sandbox policy **without** having to reach for the generic `-c/--config` option:

```shell
# Run Codex with the default, read-only sandbox
codex --sandbox read-only

# Allow the agent to write within the current workspace while still blocking network access
codex --sandbox workspace-write

# Danger! Disable sandboxing entirely (only do this if you are already running in a container or other isolated env)
codex --sandbox danger-full-access
```

The same setting can be persisted in `~/.codex/config.toml` via the top-level `sandbox_mode = "MODE"` key, e.g. `sandbox_mode = "workspace-write"`.

## Code Organization

這個 folder is the root 的 a Cargo workspace. 它 contains quite a bit 的 experimental code, but here are the key crates:

- [`core/`](./core) contains the business logic for Codex. Ultimately, we hope this to be a library crate that is generally useful for building other Rust/native applications that use Codex.
- [`exec/`](./exec) "headless" CLI for use in automation.
- [`tui/`](./tui) CLI that launches a fullscreen TUI built with [Ratatui](https://ratatui.rs/).
- [`cli/`](./cli) CLI multitool that provides the aforementioned CLIs via subcommands.
