# 設定


Codex 支援多種機制來設定組態值：

- 特定於設定的命令列旗標，例如 `--model o3`（最高優先級）。
- 一個通用的 `-c`/`--config` 旗標，可接受 `key=value` 配對，例如 `--config model="o3"`。
  - 鍵值可以包含點，以設定比根層級更深的值，例如 `--config model_providers.openai.wire_api="chat"`。
  - 值可以包含物件，例如 `--config shell_environment_policy.include_only=["PATH", "HOME", "USER"]`。
  - 為了與 `config.toml` 保持一致，值採用 TOML 格式而非 JSON 格式，因此請使用 `{a = 1, b = 2}` 而非 `{"a": 1, "b": 2}`。
  - 如果 `value` 無法被解析為有效的 TOML 值，它將被視為字串值。這意味著 `-c model="o3"` 和 `-c model=o3` 是等效的。
- `$CODEX_HOME/config.toml` 設定檔，其中 `CODEX_HOME` 環境變數值預設為 `~/.codex`。（請注意，`CODEX_HOME` 也將是儲存日誌和其他 Codex 相關資訊的地方。）

`--config` 旗標和 `config.toml` 檔案都支援以下選項：

## 模型

Codex 應使用的模型。

```toml
model = "o3"  # overrides the default of "gpt-5"
```

## 模型供應商

此選項可讓您覆寫和修改與 Codex 捆綁的預設模型供應商集。此值是一個映射，其中鍵值是與 `model_provider` 一起使用的值，用以選擇對應的供應商。

例如，如果您想新增一個透過聊天完成 API 使用 OpenAI 4o 模型的供應商，那麼您可以新增以下設定：

```toml
# Recall that in TOML, root keys must be listed before tables.
model = "gpt-4o"
model_provider = "openai-chat-completions"

[model_providers.openai-chat-completions]
# Name of the provider that will be displayed in the Codex UI.
name = "OpenAI using Chat Completions"
# The path `/chat/completions` will be amended to this URL to make the POST
# request for the chat completions.
base_url = "https://api.openai.com/v1"
# If `env_key` is set, identifies an environment variable that must be set when
# using Codex with this provider. The value of the environment variable must be
# non-empty and will be used in the `Bearer TOKEN` HTTP header for the POST request.
env_key = "OPENAI_API_KEY"
# Valid values for wire_api are "chat" and "responses". Defaults to "chat" if omitted.
wire_api = "chat"
# If necessary, extra query params that need to be added to the URL.
# See the Azure example below.
query_params = {}
```

請注意，這使得能夠將 Codex CLI 與非 OpenAI 模型一起使用，只要它們使用與 OpenAI 聊天完成 API 相容的有線 API。例如，您可以定義以下供應商，以將 Codex CLI 與在本機執行的 Ollama 一起使用：

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
```

或者一個第三方供應商（為 API 金鑰使用一個不同的環境變數）：

```toml
[model_providers.mistral]
name = "Mistral"
base_url = "https://api.mistral.ai/v1"
env_key = "MISTRAL_API_KEY"
```

請注意，Azure 要求將 `api-version` 作為查詢參數傳遞，因此在定義 Azure provider 時，請務必將其指定為 `query_params` 的一部分：

```toml
[model_providers.azure]
name = "Azure"
# Make sure you set the appropriate subdomain for this URL.
base_url = "https://YOUR_PROJECT_NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"  # Or "OPENAI_API_KEY", whichever you use.
query_params = { api-version = "2025-04-01-preview" }
```

也可以設定 provider 在請求中包含額外的 HTTP 標頭。這些可以是寫死的數值 (`http_headers`) 或從環境變數讀取的值 (`env_http_headers`)：

```toml
[model_providers.example]
# name, base_url, ...

# This will add the HTTP header `X-Example-Header` with value `example-value`
# to each request to the model provider.
http_headers = { "X-Example-Header" = "example-value" }

# This will add the HTTP header `X-Example-Features` with the value of the
# `EXAMPLE_FEATURES` environment variable to each request to the model provider
# _if_ the environment variable is set and its value is non-empty.
env_http_headers = { "X-Example-Features" = "EXAMPLE_FEATURES" }
```

### 各 provider 的網路調整

以下選用設定控制**每個 model provider** 的重試行為和串流閒置逾時。它們必須在 `config.toml` 中對應的 `[model_providers.<id>]` 區塊內指定。（舊版本接受頂層金鑰；現在它們會被忽略。）

範例：

```toml
[model_providers.openai]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
# network tuning overrides (all optional; falls back to built‑in defaults)
request_max_retries = 4            # retry failed HTTP requests
stream_max_retries = 10            # retry dropped SSE streams
stream_idle_timeout_ms = 300000    # 5m idle timeout
```

#### request_max_retries

Codex 將重試失敗的 HTTP 請求到 model provider 的次數。預設為 `4`。

#### stream_max_retries

當串流回應中斷時，Codex 將嘗試重新連線的次數。預設為 `5`。

#### stream_idle_timeout_ms

在將連線視為遺失之前，Codex 在串流回應上等待活動的時間長度。預設為 `300_000` (5 分鐘)。

## model_provider

識別要從 `model_providers` 對應中使用哪個 provider。預設為 `"openai"`。您可以透過 `OPENAI_BASE_URL` 環境變數來覆寫內建 `openai` provider 的 `base_url`。

請注意，如果您覆寫 `model_provider`，那麼您可能也想覆寫
`model`。例如，如果您正在本機執行帶有 Mistral 的 ollama，
那麼除了在 `model_providers` 對應中新增條目外，您還需要在設定中新增以下內容：

```toml
model_provider = "ollama"
model = "mistral"
```

## approval_policy

決定何時應提示使用者批准 Codex 是否可以執行命令：

```toml
# Codex has hardcoded logic that defines a set of "trusted" commands.
# Setting the approval_policy to `untrusted` means that Codex will prompt the
# user before running a command not in the "trusted" set.
#
# See https://github.com/openai/codex/issues/1260 for the plan to enable
# end-users to define their own trusted commands.
approval_policy = "untrusted"
```

如果您想在命令失敗時收到通知，請使用 "on-failure"：

```toml
# If the command fails when run in the sandbox, Codex asks for permission to
# retry the command outside the sandbox.
approval_policy = "on-failure"
```

如果您希望模型一直執行，直到它決定需要向您請求提升權限為止，請使用 "on-request"：

```toml
# The model decides when to escalate
approval_policy = "on-request"
```

或者，您可以讓模型一直執行直到完成，並且永不請求以提升的權限執行命令：

```toml
# User is never prompted: if the command fails, Codex will automatically try
# something out. Note the `exec` subcommand always uses this mode.
approval_policy = "never"
```

## 設定檔

一個 _設定檔_ 是一組可以一起設定的配置值集合。可以在 `config.toml` 中定義多個設定檔，您可以指定您
想在執行時透過 `--profile` 旗標使用的設定檔。

以下是一個定義了多個設定檔的 `config.toml` 範例：

```toml
model = "o3"
approval_policy = "untrusted"
disable_response_storage = false

# Setting `profile` is equivalent to specifying `--profile o3` on the command
# line, though the `--profile` flag can still be used to override this value.
profile = "o3"

[model_providers.openai-chat-completions]
name = "OpenAI using Chat Completions"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "chat"

[profiles.o3]
model = "o3"
model_provider = "openai"
approval_policy = "never"
model_reasoning_effort = "high"
model_reasoning_summary = "detailed"

[profiles.gpt3]
model = "gpt-3.5-turbo"
model_provider = "openai-chat-completions"

[profiles.zdr]
model = "o3"
model_provider = "openai"
approval_policy = "on-failure"
disable_response_storage = true
```

使用者可以在多個層級上指定設定值。優先順序如下：

1. 自訂命令列參數，例如 `--model o3`
2. 作為設定檔的一部分，其中 `--profile` 是透過 CLI (或在設定檔本身中) 指定的
3. 作為 `config.toml` 中的條目，例如 `model = "o3"`
4. Codex CLI 隨附的預設值 (即 Codex CLI 預設為 `gpt-5`)

## model_reasoning_effort

如果選定的模型已知支援推理 (例如：`o3`、`o4-mini`、`codex-*`、`gpt-5`)，在使用 Responses API 時，預設會啟用推理。如 [OpenAI 平台文件](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#get-started-with-reasoning) 中所述，可將其設定為：

- `"minimal"`
- `"low"`
- `"medium"` (預設)
- `"high"`

注意：若要最小化推理，請選擇 `"minimal"`。

## model_reasoning_summary

如果模型名稱以 `"o"` (如 `"o3"` 或 `"o4-mini"`) 或 `"codex"` 開頭，在使用 Responses API 時，預設會啟用推理。如 [OpenAI 平台文件](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#reasoning-summaries) 中所述，可將其設定為：

- `"auto"` (預設)
- `"concise"`
- `"detailed"`

若要停用推理摘要，請在您的設定檔中將 `model_reasoning_summary` 設定為 `"none"`：

```toml
model_reasoning_summary = "none"  # disable reasoning summaries
```

## model_verbosity

在使用 Responses API 時，控制 GPT‑5 系列模型的輸出長度/詳細程度。支援的值：

- `"low"`
- `"medium"` (省略時的預設值)
- `"high"`

設定後，Codex 會在請求的 payload 中包含一個 `text` 物件並附上已設定的詳細程度，例如：`"text": { "verbosity": "low" }`。

範例：

```toml
model = "gpt-5"
model_verbosity = "low"
```

注意：這僅適用於使用 Responses API 的供應商。Chat Completions 供應商不受影響。

## model_supports_reasoning_summaries

預設情況下，`reasoning` 僅會設定在對已知支援此功能的 OpenAI 模型的請求上。若要強制在對目前模型的請求上設定 `reasoning`，您可以透過在 `config.toml` 中設定以下內容來強制此行為：

```toml
model_supports_reasoning_summaries = true
```

## sandbox_mode

Codex 會在作業系統層級的沙箱中執行由模型產生的 shell 命令。

在大多數情況下，您可以透過單一選項選擇期望的行為：

```toml
# same as `--sandbox read-only`
sandbox_mode = "read-only"
```

預設的策略是 `read-only`，這表示命令可以讀取
磁碟上的任何檔案，但寫入檔案或存取網路的嘗試將被阻擋。

一個較寬鬆的策略是 `workspace-write`。指定後，Codex 任務的目前工作目錄將是可寫入的 (在 macOS 上 `$TMPDIR` 也是如此)。請注意，CLI 預設會使用其啟動的目錄作為 `cwd`，不過這可以使用 `--cwd/-C` 來覆寫。

在 macOS (以及即將支援的 Linux) 上，所有包含 `.git/` 資料夾作為_直接子目錄_的可寫入根目錄 (包含 `cwd`)，都會將 `.git/` 資料夾設定為唯讀，而 Git 儲存庫的其餘部分則為可寫入。這表示像 `git commit` 這樣的命令預設會失敗 (因為它需要寫入 `.git/`)，並且會需要 Codex 請求權限。

```toml
# same as `--sandbox workspace-write`
sandbox_mode = "workspace-write"

# Extra settings that only apply when `sandbox = "workspace-write"`.
[sandbox_workspace_write]
# By default, the cwd for the Codex session will be writable as well as $TMPDIR
# (if set) and /tmp (if it exists). Setting the respective options to `true`
# will override those defaults.
exclude_tmpdir_env_var = false
exclude_slash_tmp = false

# Optional list of _additional_ writable roots beyond $TMPDIR and /tmp.
writable_roots = ["/Users/YOU/.pyenv/shims"]

# Allow the command being run inside the sandbox to make outbound network
# requests. Disabled by default.
network_access = false
```

若要完全停用沙箱模式，請如下指定 `danger-full-access`：

```toml
# same as `--sandbox danger-full-access`
sandbox_mode = "danger-full-access"
```

如果 Codex 是在一個提供自身沙箱機制的環境中執行（例如 Docker 容器），以致於不需要額外的沙箱機制，那麼使用此設定是合理的。

不過，如果您嘗試在不支援其原生沙箱機制的環境中使用 Codex（例如較舊的 Linux 核心或是在 Windows 上），也可能需要使用此選項。

## 批准預設

Codex 提供三種主要的批准預設：

- 唯讀：Codex 可以讀取檔案並回答問題；編輯、執行指令和網路存取需要批准。
- 自動：Codex 可以在工作區內讀取檔案、進行編輯及執行指令，無需批准；若需存取工作區外的內容或網路，則會請求批准。
- 完整存取：無需提示即可完整存取磁碟與網路；風險極高。

您可以使用 `--ask-for-approval` 和 `--sandbox` 選項，在命令列進一步自訂 Codex 的執行方式。

## mcp_servers

定義 Codex 為使用工具而可諮詢的 MCP 伺服器列表。目前僅支援透過執行程式並經由 stdio 進行通訊所啟動的伺服器。對於使用 SSE 傳輸的伺服器，請考慮使用像 [mcp-proxy](https://github.com/sparfenyuk/mcp-proxy) 這樣的轉接器。

**注意：** Codex 可能會快取來自 MCP 伺服器的工具和資源列表，以便在啟動時能將此資訊納入上下文中，而無需產生所有伺服器。這樣的設計是為了透過延遲載入 MCP 伺服器來節省資源。

此設定選項類似於 Claude 和 Cursor 在其各自的 JSON 設定檔中定義 `mcpServers` 的方式，不過由於 Codex 使用 TOML 作為其設定語言，格式略有不同。例如，以下的 JSON 設定：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "mcp-server"],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

在 `~/.codex/config.toml` 中應表示如下：

```toml
# IMPORTANT: the top-level key is `mcp_servers` rather than `mcpServers`.
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

## disable_response_storage

目前，帳戶設定為使用零資料保留 (ZDR) 的客戶必須將 `disable_response_storage` 設定為 `true`，這樣 Codex 才能使用與 ZDR 相容的 Responses API 替代方案：

```toml
disable_response_storage = true
```

## shell_environment_policy

Codex 會產生子程序 (例如，當執行助理建議的 `local_shell` 工具呼叫時)。預設情況下，它現在會將**您的完整環境**傳遞給這些子程序。您可以透過 `config.toml` 中的 **`shell_environment_policy`** 區塊來調整此行為：

```toml
[shell_environment_policy]
# inherit can be "all" (default), "core", or "none"
inherit = "core"
# set to true to *skip* the filter for `"*KEY*"` and `"*TOKEN*"`
ignore_default_excludes = false
# exclude patterns (case-insensitive globs)
exclude = ["AWS_*", "AZURE_*"]
# force-set / override values
set = { CI = "1" }
# if provided, *only* vars matching these patterns are kept
include_only = ["PATH", "HOME"]
```

| 欄位                      | 類型                       | 預設值 | 說明                                                                                                                                    |
| ------------------------- | -------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `inherit`                 | string                     | `all`   | 環境的起始範本：<br>`all` (複製完整的父環境)、`core` (`HOME`、`PATH`、`USER` 等)，或 `none` (從空環境開始)。           |
| `ignore_default_excludes` | boolean                    | `false` | 當為 `false` 時，在其他規則執行前，Codex 會移除任何**名稱**中包含 `KEY`、`SECRET` 或 `TOKEN` (不區分大小寫) 的變數。              |
| `exclude`                 | array<string>        | `[]`    | 在預設篩選器之後要捨棄的不區分大小寫 glob 模式。<br>範例：`"AWS_*"`、`"AZURE_*"`。                                           |
| `set`                     | table<string,string> | `{}`    | 明確的鍵/值覆寫或新增 – 總是優先於繼承的值。                                                                    |
| `include_only`            | array<string>        | `[]`    | 如果非空，則為模式的白名單；只有符合_其中一個_模式的變數才能在最後一步中保留下來。(通常與 `inherit = "all"` 一起使用。) |

這些模式是 **glob 風格**，而非完整的正規表示式：`*` 匹配任何
數量的字元，`?` 精確匹配一個，而字元類別如
`[A-Z]`/`[^0-9]` 也被支援。匹配總是**不區分大小寫**的。此
語法在程式碼中被記錄為 `EnvironmentVariablePattern` (請參閱
`core/src/config_types.rs`)。

如果您只需要一個乾淨的初始狀態並加上一些自訂條目，您可以這樣寫：

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
```

目前，`CODEX_SANDBOX_NETWORK_DISABLED=1` 也會被加入到環境變數中，假設網路已停用。此項無法設定。

## notify

指定一個程式，它將被執行以接收由 Codex 產生的事件通知。請注意，該程式將會收到通知引數，其格式為 JSON 字串，例如：

```json
{
  "type": "agent-turn-complete",
  "turn-id": "12345",
  "input-messages": ["Rename `foo` to `bar` and update the callsites."],
  "last-assistant-message": "Rename complete and verified `cargo build` succeeds."
}
```

 `"type"` 屬性將永遠被設定。目前，`"agent-turn-complete"` 是唯一支援的通知類型。

舉例來說，這裡有一個 Python 腳本，它會解析 JSON 並決定是否在 macOS 上使用 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 顯示桌面推播通知：

```python
#!/usr/bin/env python3

import json
import subprocess
import sys


def main() -> int:
    if len(sys.argv) != 2:
        print("Usage: notify.py <NOTIFICATION_JSON>")
        return 1

    try:
        notification = json.loads(sys.argv[1])
    except json.JSONDecodeError:
        return 1

    match notification_type := notification.get("type"):
        case "agent-turn-complete":
            assistant_message = notification.get("last-assistant-message")
            if assistant_message:
                title = f"Codex: {assistant_message}"
            else:
                title = "Codex: Turn Complete!"
            input_messages = notification.get("input_messages", [])
            message = " ".join(input_messages)
            title += message
        case _:
            print(f"not sending a push notification for: {notification_type}")
            return 0

    subprocess.check_output(
        [
            "terminal-notifier",
            "-title",
            title,
            "-message",
            message,
            "-group",
            "codex",
            "-ignoreDnD",
            "-activate",
            "com.googlecode.iterm2",
        ]
    )

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

若要讓 Codex 使用此腳本進行通知，您可以在 `~/.codex/config.toml` 中透過 `notify` 進行設定，並使用您電腦上 `notify.py` 的正確路徑：

```toml
notify = ["python3", "/Users/mbolin/.codex/notify.py"]
```

## history

預設情況下，Codex CLI 會將傳送給模型的訊息記錄在 `$CODEX_HOME/history.jsonl` 中。請注意，在 UNIX 上，檔案權限被設定為 `o600`，因此只有擁有者可以讀取和寫入。

若要停用此行為，請如下設定 `[history]`：

```toml
[history]
persistence = "none"  # "save-all" is the default value
```

## file_opener

指定用來超連結模型輸出中引用的編輯器/URI 方案。設定後，模型輸出中的檔案引用將會使用指定的 URI 方案建立超連結，讓您可以在終端機中透過 ctrl/cmd 點擊來開啟它們。

例如，如果模型輸出包含類似 `【F:/home/user/project/main.py†L42-L50】` 的參考，那麼這將被重寫為連結至 URI `vscode://file/home/user/project/main.py:42`。

請注意，這**並非**一個通用的編輯器設定（例如 `$EDITOR`），因為它只接受一組固定的值：

- `"vscode"` (預設)
- `"vscode-insiders"`
- `"windsurf"`
- `"cursor"`
- `"none"` 以明確停用此功能

目前，`"vscode"` 是預設值，但 Codex 不會驗證 VS Code 是否已安裝。因此，`file_opener` 未來可能會預設為 `"none"` 或其他值。

## hide_agent_reasoning

Codex 會間歇性地發出「推理」事件，以顯示模型在產生最終答案前的內部「思考過程」。有些使用者可能會覺得這些事件令人分心，尤其是在 CI 日誌或簡潔的終端機輸出中。

將 `hide_agent_reasoning` 設定為 `true` 會在 TUI 和無介面的 `exec` 子命令中**都**抑制這些事件：

```toml
hide_agent_reasoning = true   # defaults to false
```

## show_raw_agent_reasoning

在可用時，顯示模型的原始思維鏈（「原始推理內容」）。

注意：

- 僅當所選模型/供應商實際發出原始推理內容時才會生效。許多模型並不會發出。當不支援時，此選項沒有可見效果。
- 原始推理可能包含中間思緒或敏感內容。僅在您的工作流程可接受的情況下啟用。

範例：

```toml
show_raw_agent_reasoning = true  # defaults to false
```

## model_context_window

模型的內容視窗大小，以 token 為單位。

一般來說，Codex 知道最常見的 OpenAI 模型的內容視窗大小，但如果您正在使用一個新模型搭配舊版的 Codex CLI，那麼您可以使用 `model_context_window` 來告知 Codex 應該使用什麼值，以判斷對話期間還剩下多少內容空間。

## model_max_output_tokens

這與 `model_context_window` 類似，但適用於模型的最大輸出 token 數量。

## project_doc_max_bytes

從 `AGENTS.md` 檔案讀取的最大位元組數，以包含在工作階段第一輪傳送的指令中。預設為 32 KiB。

## tui

TUI 專屬的選項。

```toml
[tui]
# More to come here
```

## 設定參考

| 鍵 | 類型 / 值 | 說明 |
| --- | --- | --- |
| `model` | string | 要使用的模型（例如 `gpt-5`）。 |
| `model_provider` | string | 來自 `model_providers` 的供應商 ID（預設：`openai`）。 |
| `model_context_window` | number | 上下文視窗的 token 數量。 |
| `model_max_output_tokens` | number | 最大輸出 token 數量。 |
| `approval_policy` | `untrusted` \| `on-failure` \| `on-request` \| `never` | 何時提示需要批准。 |
| `sandbox_mode` | `read-only` \| `workspace-write` \| `danger-full-access` | 作業系統沙箱政策。 |
| `sandbox_workspace_write.writable_roots` | array<string> | 在 workspace‑write 模式中額外可寫入的根目錄。 |
| `sandbox_workspace_write.network_access` | boolean | 在 workspace‑write 模式中允許網路存取（預設：false）。 |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean | 從可寫入根目錄中排除 `$TMPDIR`（預設：false）。 |
| `sandbox_workspace_write.exclude_slash_tmp` | boolean | 從可寫入根目錄中排除 `/tmp`（預設：false）。 |
| `disable_response_storage` | boolean | ZDR 組織的必要設定。 |
| `notify` | array<string> | 用於通知的外部程式。 |
| `instructions` | string | 目前忽略；請使用 `experimental_instructions_file` 或 `AGENTS.md`。 |
| `mcp_servers.<id>.command` | string | MCP 伺服器啟動器命令。 |
| `mcp_servers.<id>.args` | array<string> | MCP 伺服器參數。 |
| `mcp_servers.<id>.env` | map<string,string> | MCP 伺服器環境變數。 |
| `model_providers.<id>.name` | string | 顯示名稱。 |
| `model_providers.<id>.base_url` | string | API 基礎 URL。 |
| `model_providers.<id>.env_key` | string | 用於 API 金鑰的環境變數。 |
| `model_providers.<id>.wire_api` | `chat` \| `responses` | 使用的協定（預設：`chat`）。 |
| `model_providers.<id>.query_params` | map<string,string> | 額外的查詢參數（例如 Azure 的 `api-version`）。 |
| `model_providers.<id>.http_headers` | map<string,string> | 額外的靜態標頭。 |
| `model_providers.<id>.env_http_headers` | map<string,string> | 從環境變數取得的標頭。 |
| `model_providers.<id>.request_max_retries` | number | 每個供應商的 HTTP 重試次數（預設：4）。 |
| `model_providers.<id>.stream_max_retries` | number | SSE 串流重試次數（預設：5）。 |
| `model_providers.<id>.stream_idle_timeout_ms` | number | SSE 閒置逾時（毫秒）（預設：300000）。 |
| `project_doc_max_bytes` | number | 從 `AGENTS.md` 讀取的最大位元組數。 |
| `profile` | string | 當前啟用的設定檔名稱。 |
| `profiles.<name>.*` | various | 設定檔範圍內對相同鍵值的覆寫。 |
| `history.persistence` | `save-all` \| `none` | 歷史記錄檔案的持續性（預設：`save-all`）。 |
| `history.max_bytes` | number | 目前忽略（未強制執行）。 |
| `file_opener` | `vscode` \| `vscode-insiders` \| `windsurf` \| `cursor` \| `none` | 可點擊引用的 URI 配置（預設：`vscode`）。 |
| `tui` | table | TUI 專用選項（保留）。 |
| `hide_agent_reasoning` | boolean | 隱藏模型推理事件。 |
| `show_raw_agent_reasoning` | boolean | 顯示原始推理過程（如果可用）。 |
| `model_reasoning_effort` | `minimal` \| `low` \| `medium` \| `high` | Responses API 的推理程度。 |
| `model_reasoning_summary` | `auto` \| `concise` \| `detailed` \| `none` | 推理摘要。 |
| `model_verbosity` | `low` \| `medium` \| `high` | GPT-5 文字詳細程度（Responses API）。 |
| `model_supports_reasoning_summaries` | boolean | 強制啟用推理摘要。 |
| `chatgpt_base_url` | string | ChatGPT 驗證流程的基礎 URL。 |
| `experimental_resume` | string (path) | 繼續執行的 JSONL 路徑（內部/實驗性）。 |
| `experimental_instructions_file` | string (path) | 取代內建指令（實驗性）。 |
| `experimental_use_exec_command_tool` | boolean | 使用實驗性的 exec 命令工具。 |
| `use_experimental_reasoning_summary` | boolean | 為推理鏈使用實驗性摘要。 |
| `responses_originator_header_internal_override` | string | 覆寫 `originator` 標頭值。 |
| `projects.<path>.trust_level` | string | 將專案/工作樹標記為受信任（僅辨識 `"trusted"`）。 |
| `preferred_auth_method` | `chatgpt` \| `apikey` | 選擇預設驗證方法（預設：`chatgpt`）。 |
| `tools.web_search` | boolean | 啟用網路搜尋工具（別名：`web_search_request`）（預設：false）。 |
