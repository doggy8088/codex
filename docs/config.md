# 設定


Codex 支援數種設定值的機制：

- 特定設定的命令列旗標，如 `--model o3`（最高優先順序）。
- 通用的 `-c`/`--config` 旗標，接受 `key=value` 對，如 `--config model="o3"`。
  - 金鑰可以包含點號以設定比根層級更深的值，例如 `--config model_providers.openai.wire_api="chat"`。
  - 值可以包含物件，如 `--config shell_environment_policy.include_only=["PATH", "HOME", "USER"]`。
  - 為了與 `config.toml` 保持一致，值採用 TOML 格式而非 JSON 格式，因此使用 `{a = 1, b = 2}` 而非 `{"a": 1, "b": 2}`。
  - 如果 `value` 無法解析為有效的 TOML 值，則被視為字串值。這意味著 `-c model="o3"` 和 `-c model=o3` 是等效的。
- `$CODEX_HOME/config.toml` 設定檔，其中 `CODEX_HOME` 環境值預設為 `~/.codex`。（注意 `CODEX_HOME` 也是記錄檔和其他 Codex 相關資訊的儲存位置。）

`--config` 旗標和 `config.toml` 檔案都支援以下選項：

## model

Codex 應該使用的模型。

```toml
model = "o3"  # 覆蓋預設的 "gpt-5"
```

## model_providers

此選項讓您覆蓋和修改 Codex 內建的預設模型提供者集合。此值是一個對應表，其中金鑰是與 `model_provider` 一起使用以選擇對應提供者的值。

例如，如果您想新增一個透過 chat completions API 使用 OpenAI 4o 模型的提供者，您可以新增以下設定：

```toml
# 請記住，在 TOML 中，根金鑰必須列在表格之前。
model = "gpt-4o"
model_provider = "openai-chat-completions"

[model_providers.openai-chat-completions]
# 將在 Codex UI 中顯示的提供者名稱。
name = "OpenAI using Chat Completions"
# 路徑 `/chat/completions` 將附加到此 URL 以進行 chat completions 的 POST 請求。
base_url = "https://api.openai.com/v1"
# 如果設定了 `env_key`，則識別在使用此提供者的 Codex 時必須設定的環境變數。
# 環境變數的值必須非空，並將用於 POST 請求的 `Bearer TOKEN` HTTP 標頭。
env_key = "OPENAI_API_KEY"
# wire_api 的有效值為 "chat" 和 "responses"。如果省略則預設為 "chat"。
wire_api = "chat"
# 如有必要，需要新增到 URL 的額外查詢參數。
# 請參閱下方的 Azure 範例。
query_params = {}
```

注意這使得 Codex CLI 可以與非 OpenAI 模型一起使用，只要它們使用與 OpenAI chat completions API 相容的 wire API。例如，您可以定義以下提供者以在本機執行的 Ollama 中使用 Codex CLI：

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
```

或第三方提供者（為 API 金鑰使用不同的環境變數）：

```toml
[model_providers.mistral]
name = "Mistral"
base_url = "https://api.mistral.ai/v1"
env_key = "MISTRAL_API_KEY"
```

注意 Azure 需要將 `api-version` 作為查詢參數傳遞，因此在定義 Azure 提供者時請確保將其指定為 `query_params` 的一部分：

```toml
[model_providers.azure]
name = "Azure"
# 請確保為此 URL 設定適當的子網域。
base_url = "https://YOUR_PROJECT_NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"  # 或 "OPENAI_API_KEY"，無論您使用哪個。
query_params = { api-version = "2025-04-01-preview" }
```

也可以設定提供者在請求中包含額外的 HTTP 標頭。這些可以是硬編碼值（`http_headers`）或從環境變數讀取的值（`env_http_headers`）：

```toml
[model_providers.example]
# name, base_url, ...

# 這將在每個到模型提供者的請求中新增值為 `example-value` 的 HTTP 標頭 `X-Example-Header`。
http_headers = { "X-Example-Header" = "example-value" }

# 這將在每個到模型提供者的請求中新增值為 `EXAMPLE_FEATURES` 環境變數值的 HTTP 標頭 `X-Example-Features`，
# _如果_ 環境變數已設定且其值非空。
env_http_headers = { "X-Example-Features" = "EXAMPLE_FEATURES" }
```

### 每個提供者的網路調整

以下可選設定控制 **每個模型提供者** 的重試行為和串流空閒逾時。它們必須在 `config.toml` 中的對應 `[model_providers.<id>]` 區塊內指定。（較舊版本接受頂層金鑰；現在被忽略。）

範例：

```toml
[model_providers.openai]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
# 網路調整覆蓋（全部可選；回退到內建預設值）
request_max_retries = 4            # 重試失敗的 HTTP 請求
stream_max_retries = 10            # 重試斷開的 SSE 串流
stream_idle_timeout_ms = 300000    # 5 分鐘空閒逾時
```

#### request_max_retries

Codex 對模型提供者重試失敗 HTTP 請求的次數。預設為 `4`。

#### stream_max_retries

當串流回應中斷時，Codex 嘗試重新連接的次數。預設為 `5`。

#### stream_idle_timeout_ms

Codex 在將連接視為遺失之前等待串流回應活動的時間。預設為 `300_000`（5 分鐘）。

## model_provider

從 `model_providers` 對應表中識別要使用的提供者。預設為 `"openai"`。您可以透過 `OPENAI_BASE_URL` 環境變數覆蓋內建 `openai` 提供者的 `base_url`。

注意如果您覆蓋 `model_provider`，那麼您可能也想覆蓋 `model`。例如，如果您在本機執行帶有 Mistral 的 ollama，那麼除了在 `model_providers` 對應表中新增條目外，您還需要將以下內容新增到您的設定中：

```toml
model_provider = "ollama"
model = "mistral"
```

## approval_policy

決定何時應提示使用者核准 Codex 是否可以執行命令：

```toml
# Codex 有硬編碼邏輯定義一組「受信任」命令。
# 將 approval_policy 設定為 `untrusted` 意味著 Codex 將在執行不在「受信任」集合中的命令之前提示使用者。
#
# 請參閱 https://github.com/openai/codex/issues/1260 了解讓終端使用者定義自己受信任命令的計劃。
approval_policy = "untrusted"
```

如果您想在命令失敗時收到通知，請使用 "on-failure"：

```toml
# 如果命令在沙盒中執行失敗，Codex 會請求許可在沙盒外重試命令。
approval_policy = "on-failure"
```

如果您希望模型執行直到它決定需要向您請求升級權限，請使用 "on-request"：

```toml
# 模型決定何時升級
approval_policy = "on-request"
```

或者，您可以讓模型執行直到完成，並且永不請求以升級權限執行命令：

```toml
# 永不提示使用者：如果命令失敗，Codex 會自動嘗試其他方法。
# 注意 `exec` 子命令總是使用此模式。
approval_policy = "never"
```

## profiles

_設定檔_ 是可以一起設定的設定值集合。可以在 `config.toml` 中定義多個設定檔，您可以透過 `--profile` 旗標在執行時指定要使用的設定檔。

以下是定義多個設定檔的 `config.toml` 範例：

```toml
model = "o3"
approval_policy = "untrusted"
disable_response_storage = false

# 設定 `profile` 等同於在命令列上指定 `--profile o3`，
# 儘管 `--profile` 旗標仍可用於覆蓋此值。
profile = "o3"

[model_providers.openai-chat-completions]
name = "OpenAI using Chat Completions"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
wire_api = "chat"

[profiles.o3]
model = "o3"
model_provider = "openai"
approval_policy = "never"model_reasoning_effort = "high"
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

使用者可以在多個層級指定設定值。優先順序如下：

1. 自訂命令列參數，例如 `--model o3`
2. 作為設定檔的一部分，其中 `--profile` 透過 CLI 指定（或在設定檔本身中）
3. 作為 `config.toml` 中的條目，例如 `model = "o3"`
4. Codex CLI 預設值（即 Codex CLI 預設為 `gpt-5`）

## model_reasoning_effort

如果選定的模型已知支援推理（例如：`o3`、`o4-mini`、`codex-*`、`gpt-5`），在使用 Responses API 時預設會啟用推理。如 [OpenAI Platform 文件](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#get-started-with-reasoning) 所述，這可以設定為：

- `"minimal"`
- `"low"`
- `"medium"`（預設）
- `"high"`

注意：要最小化推理，請選擇 `"minimal"`。

## model_reasoning_summary

如果模型名稱以 `"o"`（如 `"o3"` 或 `"o4-mini"`）或 `"codex"` 開頭，在使用 Responses API 時預設會啟用推理。如 [OpenAI Platform 文件](https://platform.openai.com/docs/guides/reasoning?api-mode=responses#reasoning-summaries) 所述，這可以設定為：

- `"auto"`（預設）
- `"concise"`
- `"detailed"`

要停用推理摘要，請在您的設定中將 `model_reasoning_summary` 設定為 `"none"`：

```toml
model_reasoning_summary = "none"  # 停用推理摘要
```

## model_verbosity

在使用 Responses API 時控制 GPT‑5 系列模型的輸出長度/詳細程度。支援的值：

- `"low"`
- `"medium"`（省略時的預設值）
- `"high"`

設定時，Codex 會在請求負載中包含具有設定詳細程度的 `text` 物件，例如：`"text": { "verbosity": "low" }`。

範例：

```toml
model = "gpt-5"
model_verbosity = "low"
```

注意：這僅適用於使用 Responses API 的提供者。Chat Completions 提供者不受影響。

## model_supports_reasoning_summaries

預設情況下，`reasoning` 僅在對已知支援的 OpenAI 模型請求時設定。要強制在對目前模型的請求中設定 `reasoning`，您可以透過在 `config.toml` 中設定以下內容來強制此行為：

```toml
model_supports_reasoning_summaries = true
```

## sandbox_mode

Codex 在作業系統級沙盒內執行模型生成的 shell 命令。

在大多數情況下，您可以用單一選項選擇所需的行為：

```toml
# 與 `--sandbox read-only` 相同
sandbox_mode = "read-only"
```

預設政策是 `read-only`，這意味著命令可以讀取磁碟上的任何檔案，但嘗試寫入檔案或存取網路將被阻止。

更寬鬆的政策是 `workspace-write`。指定時，Codex 任務的目前工作目錄將是可寫的（以及 macOS 上的 `$TMPDIR`）。注意 CLI 預設使用其被執行的目錄作為 `cwd`，儘管這可以使用 `--cwd/-C` 覆蓋。

在 macOS（以及即將在 Linux）上，所有包含 `.git/` 資料夾 _作為直接子項_ 的可寫根目錄（包括 `cwd`）將配置 `.git/` 資料夾為唯讀，而 Git 儲存庫的其餘部分將是可寫的。這意味著 `git commit` 等命令預設會失敗（因為它需要寫入 `.git/`），並需要 Codex 請求許可。

```toml
# 與 `--sandbox workspace-write` 相同
sandbox_mode = "workspace-write"

# 僅在 `sandbox = "workspace-write"` 時適用的額外設定。
[sandbox_workspace_write]
# 預設情況下，Codex 會話的 cwd 將是可寫的，以及 $TMPDIR（如果設定）和 /tmp（如果存在）。
# 將相應選項設定為 `true` 將覆蓋這些預設值。
exclude_tmpdir_env_var = false
exclude_slash_tmp = false

# 除了 $TMPDIR 和 /tmp 之外的 _額外_ 可寫根目錄的可選列表。
writable_roots = ["/Users/YOU/.pyenv/shims"]

# 允許在沙盒內執行的命令進行對外網路請求。預設停用。
network_access = false
```

要完全停用沙盒，請如此指定 `danger-full-access`：

```toml
# 與 `--sandbox danger-full-access` 相同
sandbox_mode = "danger-full-access"
```

如果 Codex 在提供自己沙盒的環境（如 Docker 容器）中執行，使得進一步的沙盒不必要，這是合理的使用方式。

如果您嘗試在不支援其原生沙盒機制的環境中使用 Codex（如較舊的 Linux 核心或 Windows 上），使用此選項可能也是必要的。

## 核准預設

Codex 提供三種主要的核准預設：

- 唯讀：Codex 可以讀取檔案並回答問題；編輯、執行命令和網路存取需要核准。
- 自動：Codex 可以讀取檔案、進行編輯，並在工作空間中執行命令而無需核准；在工作空間外或網路存取時請求核准。
- 完全存取：完全磁碟和網路存取而無提示；極度危險。

您可以使用 `--ask-for-approval` 和 `--sandbox` 選項在命令列進一步自訂 Codex 的執行方式。

## mcp_servers

定義 Codex 可以諮詢工具使用的 MCP 伺服器列表。目前，僅支援透過執行程式通過 stdio 通訊啟動的伺服器。對於使用 SSE 傳輸的伺服器，請考慮使用如 [mcp-proxy](https://github.com/sparfenyuk/mcp-proxy) 等適配器。

**注意：** Codex 可能會快取來自 MCP 伺服器的工具和資源列表，以便 Codex 可以在啟動時將此資訊包含在上下文中而無需產生所有伺服器。這旨在透過延遲載入 MCP 伺服器來節省資源。

此設定選項與 Claude 和 Cursor 在其各自 JSON 設定檔中定義 `mcpServers` 的方式相當，儘管因為 Codex 使用 TOML 作為其設定語言，格式略有不同。例如，JSON 中的以下設定：

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

應在 `~/.codex/config.toml` 中表示如下：

```toml
# 重要：頂層金鑰是 `mcp_servers` 而非 `mcpServers`。
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

## disable_response_storage

目前，帳戶設定為使用零資料保留 (ZDR) 的客戶必須將 `disable_response_storage` 設定為 `true`，以便 Codex 使用與 ZDR 相容的 Responses API 替代方案：

```toml
disable_response_storage = true
```

## shell_environment_policy

Codex 產生子程序（例如當執行助理建議的 `local_shell` 工具呼叫時）。預設情況下，它現在將 **您的完整環境** 傳遞給這些子程序。您可以透過 `config.toml` 中的 **`shell_environment_policy`** 區塊調整此行為：

```toml
[shell_environment_policy]
# inherit 可以是 "all"（預設）、"core" 或 "none"
inherit = "core"
# 設定為 true 以 *跳過* 對 `"*KEY*"` 和 `"*TOKEN*"` 的過濾器
ignore_default_excludes = false
# 排除模式（不區分大小寫的萬用字元）
exclude = ["AWS_*", "AZURE_*"]
# 強制設定 / 覆蓋值
set = { CI = "1" }
# 如果提供，*僅* 符合這些模式的變數被保留
include_only = ["PATH", "HOME"]
```

| 欄位                      | 類型                       | 預設    | 描述                                                                                                                                            |
| ------------------------- | -------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `inherit`                 | string                     | `all`   | 環境的起始範本：<br>`all`（複製完整父環境）、`core`（`HOME`、`PATH`、`USER`…）或 `none`（空白開始）。                                        |
| `ignore_default_excludes` | boolean                    | `false` | 當 `false` 時，Codex 會在其他規則執行前移除任何 **名稱** 包含 `KEY`、`SECRET` 或 `TOKEN`（不區分大小寫）的變數。                            |
| `exclude`                 | array<string>        | `[]`    | 在預設過濾器後要丟棄的不區分大小寫萬用字元模式。<br>範例：`"AWS_*"`、`"AZURE_*"`。                                                             |
| `set`                     | table<string,string> | `{}`    | 明確的金鑰/值覆蓋或新增 - 總是勝過繼承的值。                                                                                                   |
| `include_only`            | array<string>        | `[]`    | 如果非空，為模式白名單；只有符合 _一個_ 模式的變數能在最後步驟存活。（通常與 `inherit = "all"` 一起使用。）                                    |

模式是 **萬用字元風格**，不是完整的正規表示式：`*` 符合任何數量的字元，`?` 恰好符合一個，以及字元類別如 `[A-Z]`/`[^0-9]` 都被支援。匹配總是 **不區分大小寫**。此語法在程式碼中記錄為 `EnvironmentVariablePattern`（參見 `core/src/config_types.rs`）。

如果您只需要一個包含少數自訂條目的乾淨環境，您可以撰寫：

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
```

目前，假設網路被停用，`CODEX_SANDBOX_NETWORK_DISABLED=1` 也會被新增到環境中。這不可設定。

## notify

指定將執行以獲得 Codex 生成事件通知的程式。注意程式將接收通知參數作為 JSON 字串，例如：

```json
{
  "type": "agent-turn-complete",
  "turn-id": "12345",
  "input-messages": ["Rename `foo` to `bar` and update the callsites."],
  "last-assistant-message": "Rename complete and verified `cargo build` succeeds."
}
```

`"type"` 屬性將總是被設定。目前，`"agent-turn-complete"` 是唯一支援的通知類型。

作為範例，以下是一個 Python 腳本，它解析 JSON 並決定是否在 macOS 上使用 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 顯示桌面推送通知：

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

要讓 Codex 使用此腳本進行通知，您需要在 `~/.codex/config.toml` 中透過 `notify` 設定，使用您電腦上 `notify.py` 的適當路徑：

```toml
notify = ["python3", "/Users/mbolin/.codex/notify.py"]
```

## history

預設情況下，Codex CLI 在 `$CODEX_HOME/history.jsonl` 中記錄發送到模型的訊息。注意在 UNIX 上，檔案權限設定為 `o600`，因此它應該只能被擁有者讀取和寫入。

要停用此行為，請如下設定 `[history]`：

```toml
[history]
persistence = "none"  # "save-all" 是預設值
```

## file_opener

識別用於模型輸出中引用超連結的編輯器/URI 方案。如果設定，模型輸出中的檔案引用將使用指定的 URI 方案進行超連結，以便可以從終端中 ctrl/cmd-點擊開啟它們。

例如，如果模型輸出包含如 `【F:/home/user/project/main.py†L42-L50】` 的引用，那麼這將被重寫為連結到 URI `vscode://file/home/user/project/main.py:42`。

注意這 **不是** 一般的編輯器設定（如 `$EDITOR`），因為它只接受一組固定的值：

- `"vscode"`（預設）
- `"vscode-insiders"`
- `"windsurf"`
- `"cursor"`
- `"none"` 明確停用此功能

目前，`"vscode"` 是預設值，儘管 Codex 不會驗證 VS Code 是否已安裝。因此，`file_opener` 未來可能預設為 `"none"` 或其他值。

## hide_agent_reasoning

Codex 間歇性地發出「推理」事件，顯示模型在產生最終答案前的內部「思考」。某些使用者可能發現這些事件會分散注意力，特別是在 CI 記錄檔或最小終端輸出中。

將 `hide_agent_reasoning` 設定為 `true` 會在 **TUI** 以及無頭 `exec` 子命令中抑制這些事件：

```toml
hide_agent_reasoning = true   # 預設為 false
```

## show_raw_agent_reasoning

在可用時顯示模型的原始思維鏈（「原始推理內容」）。

注意事項：

- 僅在選定的模型/提供者實際發出原始推理內容時生效。許多模型不會。當不支援時，此選項沒有可見效果。
- 原始推理可能包含中間思想或敏感上下文。僅在您的工作流程可接受時啟用。

範例：

```toml
show_raw_agent_reasoning = true  # 預設為 false
```

## model_context_window

模型的上下文視窗大小，以 token 為單位。

一般來說，Codex 知道最常見 OpenAI 模型的上下文視窗，但如果您使用舊版 Codex CLI 搭配新模型，那麼您可以使用 `model_context_window` 告訴 Codex 在對話期間要使用什麼值來決定剩餘多少上下文。

## model_max_output_tokens

這類似於 `model_context_window`，但用於模型的最大輸出 token 數。

## project_doc_max_bytes

從 `AGENTS.md` 檔案讀取並包含在會話第一輪發送的指示中的最大位元組數。預設為 32 KiB。

## tui

TUI 特定的選項。

```toml
[tui]
# 這裡會有更多內容
```

## 設定參考

| 金鑰 | 類型 / 值 | 注意事項 |
| --- | --- | --- |
| `model` | string | 要使用的模型（例如 `gpt-5`）。 |
| `model_provider` | string | 來自 `model_providers` 的提供者 id（預設：`openai`）。 |
| `model_context_window` | number | 上下文視窗 token。 |
| `model_max_output_tokens` | number | 最大輸出 token。 |
| `approval_policy` | `untrusted` \| `on-failure` \| `on-request` \| `never` | 何時提示核准。 |
| `sandbox_mode` | `read-only` \| `workspace-write` \| `danger-full-access` | 作業系統沙盒政策。 |
| `sandbox_workspace_write.writable_roots` | array<string> | workspace‑write 中的額外可寫根目錄。 |
| `sandbox_workspace_write.network_access` | boolean | 在 workspace‑write 中允許網路（預設：false）。 |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean | 從可寫根目錄中排除 `$TMPDIR`（預設：false）。 |
| `sandbox_workspace_write.exclude_slash_tmp` | boolean | 從可寫根目錄中排除 `/tmp`（預設：false）。 |
| `disable_response_storage` | boolean | ZDR 組織必需。 |
| `notify` | array<string> | 通知的外部程式。 |
| `instructions` | string | 目前被忽略；使用 `experimental_instructions_file` 或 `AGENTS.md`。 |
| `mcp_servers.<id>.command` | string | MCP 伺服器啟動器命令。 |
| `mcp_servers.<id>.args` | array<string> | MCP 伺服器參數。 |
| `mcp_servers.<id>.env` | map<string,string> | MCP 伺服器環境變數。 |
| `model_providers.<id>.name` | string | 顯示名稱。 |
| `model_providers.<id>.base_url` | string | API 基礎 URL。 |
| `model_providers.<id>.env_key` | string | API 金鑰的環境變數。 |
| `model_providers.<id>.wire_api` | `chat` \| `responses` | 使用的協定（預設：`chat`）。 |
| `model_providers.<id>.query_params` | map<string,string> | 額外的查詢參數（例如 Azure `api-version`）。 |
| `model_providers.<id>.http_headers` | map<string,string> | 額外的靜態標頭。 |
| `model_providers.<id>.env_http_headers` | map<string,string> | 來自環境變數的標頭。 |
| `model_providers.<id>.request_max_retries` | number | 每個提供者的 HTTP 重試次數（預設：4）。 |
| `model_providers.<id>.stream_max_retries` | number | SSE 串流重試次數（預設：5）。 |
| `model_providers.<id>.stream_idle_timeout_ms` | number | SSE 空閒逾時（毫秒）（預設：300000）。 |
| `project_doc_max_bytes` | number | 從 `AGENTS.md` 讀取的最大位元組數。 |
| `profile` | string | 活動設定檔名稱。 |
| `profiles.<name>.*` | various | 相同金鑰的設定檔範圍覆蓋。 |
| `history.persistence` | `save-all` \| `none` | 歷史檔案持久化（預設：`save-all`）。 |
| `history.max_bytes` | number | 目前被忽略（未強制執行）。 |
| `file_opener` | `vscode` \| `vscode-insiders` \| `windsurf` \| `cursor` \| `none` | 可點擊引用的 URI 方案（預設：`vscode`）。 |
| `tui` | table | TUI‑特定選項（保留）。 |
| `hide_agent_reasoning` | boolean | 隱藏模型推理事件。 |
| `show_raw_agent_reasoning` | boolean | 顯示原始推理（當可用時）。 |
| `model_reasoning_effort` | `minimal` \| `low` \| `medium` \| `high` | Responses API 推理努力。 |
| `model_reasoning_summary` | `auto` \| `concise` \| `detailed` \| `none` | 推理摘要。 |
| `model_verbosity` | `low` \| `medium` \| `high` | GPT‑5 文字詳細程度（Responses API）。 |
| `model_supports_reasoning_summaries` | boolean | 強制啟用推理摘要。 |
| `chatgpt_base_url` | string | ChatGPT 認證流程的基礎 URL。 |
| `experimental_resume` | string (path) | 恢復 JSONL 路徑（內部/實驗性）。 |
| `experimental_instructions_file` | string (path) | 替換內建指示（實驗性）。 |
| `experimental_use_exec_command_tool` | boolean | 使用實驗性執行命令工具。 |
| `use_experimental_reasoning_summary` | boolean | 對推理鏈使用實驗性摘要。 |
| `responses_originator_header_internal_override` | string | 覆蓋 `originator` 標頭值。 |
| `projects.<path>.trust_level` | string | 將專案/工作樹標記為受信任（僅識別 `"trusted"`）。 |
| `preferred_auth_method` | `chatgpt` \| `apikey` | 選擇預設認證方法（預設：`chatgpt`）。 |
| `tools.web_search` | boolean | 啟用網頁搜尋工具（別名：`web_search_request`）（預設：false）。 |