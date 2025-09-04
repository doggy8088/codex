# 設定


Codex 提供多種方式設定組態值：

- 具體參數的命令列旗標，例如 `--model o3`（優先序最高）。
- 通用的 `-c`/`--config` 旗標，接受 `key=value` 配對，例如 `--config model="o3"`。
  - key 可使用點號指定更深層級，例如 `--config model_providers.openai.wire_api="chat"`。
  - value 可為物件，例如 `--config shell_environment_policy.include_only=["PATH", "HOME", "USER"]`。
  - 為與 `config.toml` 一致，value 採用 TOML 格式而非 JSON，所以請使用 `{a = 1, b = 2}`，而非 `{"a": 1, "b": 2}`。
  - 若 `value` 不是有效的 TOML 值，將被視為字串處理。亦即 `-c model="o3"` 與 `-c model=o3` 等效。
- `$CODEX_HOME/config.toml` 組態檔，其中 `CODEX_HOME` 環境變數預設為 `~/.codex`。（注意 `CODEX_HOME` 也是記錄檔與其他 Codex 相關資料的儲存位置。）

`--config` 旗標與 `config.toml` 檔案皆支援以下選項：

## model

指定 Codex 要使用的模型。

```toml
model = "o3"  # 覆寫預設值 "gpt-5"
```

## model_providers

此選項可覆寫與擴充 Codex 內建的模型供應商清單。其為一個對映，key 將與 `model_provider` 的值相對應，用來選擇特定供應商。

例如，若想新增一個透過 Chat Completions API 使用 OpenAI 4o 模型的供應商，可加入以下設定：

```toml
# 請注意，在 TOML 中，頂層 key 需寫在 table 之前。
model = "gpt-4o"
model_provider = "openai-chat-completions"

[model_providers.openai-chat-completions]
# 將顯示於 Codex UI 的供應商名稱。
name = "OpenAI using Chat Completions"
# 會在此 URL 後加上 `/chat/completions` 路徑來發出 Chat Completions 的 POST 請求。
base_url = "https://api.openai.com/v1"
# 若設定 `env_key`，代表使用此供應商時必須存在的環境變數。
# 其值不可為空，並會用於 POST 請求的 `Bearer TOKEN` HTTP 標頭。
env_key = "OPENAI_API_KEY"
# `wire_api` 可為 "chat" 或 "responses"。未指定時預設為 "chat"。
wire_api = "chat"
# 若有需要，可在此加入額外查詢參數。
# 例如下方的 Azure 範例。
query_params = {}
```

注意：只要其 wire API 與 OpenAI Chat Completions API 相容，便可透過此設定讓 Codex CLI 使用非 OpenAI 的模型。例如，您可定義以下供應商，以便在本機執行的 Ollama 上搭配 Codex CLI：

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
```

或是第三方供應商（API 金鑰使用不同的環境變數）：

```toml
[model_providers.mistral]
name = "Mistral"
base_url = "https://api.mistral.ai/v1"
env_key = "MISTRAL_API_KEY"
```

請注意：Azure 需要透過查詢參數傳入 `api-version`，因此在定義 Azure 供應商時，務必於 `query_params` 指定：

```toml
[model_providers.azure]
name = "Azure"
# Make sure you set the appropriate subdomain for this URL.
base_url = "https://YOUR_PROJECT_NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"  # Or "OPENAI_API_KEY", whichever you use.
query_params = { api-version = "2025-04-01-preview" }
```

也可設定供應商在請求中加入額外的 HTTP 標頭。其可為硬編碼值（`http_headers`），或從環境變數讀取（`env_http_headers`）：

```toml
[model_providers.example]
# name, base_url, ...

# 這會在送往模型供應商的每個請求中，加入 `X-Example-Header: example-value`。
http_headers = { "X-Example-Header" = "example-value" }

# 若環境變數已設定且非空，這會將 `EXAMPLE_FEATURES` 的值
# 作為 `X-Example-Features` 標頭加入每個請求。
env_http_headers = { "X-Example-Features" = "EXAMPLE_FEATURES" }
```

### 依供應商調校網路行為

以下可選設定可調整「每個模型供應商」的重試行為與串流閒置逾時。請將其放在 `config.toml` 的對應 `[model_providers.<id>]` 區塊中。（舊版可使用頂層 key，現已忽略。）

範例：

```toml
[model_providers.openai]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
env_key = "OPENAI_API_KEY"
# 覆寫網路調校（全為可選；未設定時採內建預設值）
request_max_retries = 4            # 失敗 HTTP 請求的重試次數
stream_max_retries = 10            # 中斷的 SSE 串流重連次數
stream_idle_timeout_ms = 300000    # 串流閒置逾時 5 分鐘
```

#### request_max_retries

Codex 在 HTTP 請求失敗時對模型供應商重試的次數。預設為 `4`。

#### stream_max_retries

串流回應中斷時，Codex 嘗試重連的次數。預設為 `5`。

#### stream_idle_timeout_ms

在串流回應無活動時，Codex 視為連線遺失前等待的時間。預設 `300_000`（5 分鐘）。

## model_provider

指定從 `model_providers` 對映中要使用的供應商。預設為 `"openai"`。您可透過 `OPENAI_BASE_URL` 環境變數覆寫內建 `openai` 供應商的 `base_url`。

注意：若您覆寫了 `model_provider`，通常也需要一併覆寫 `model`。例如，若您在本機以 ollama 執行 Mistral，除了在 `model_providers` 對映新增條目外，還需在組態加入：

```toml
model_provider = "ollama"
model = "mistral"
```

## approval_policy

決定何時提示使用者核准 Codex 是否可執行指令：

```toml
# Codex 內建一組「信任的」指令清單。
# 設為 `untrusted` 表示執行不在清單內的指令前，會先提示使用者核准。
#
# 讓最終使用者可自訂信任指令的計畫，請見：
# https://github.com/openai/codex/issues/1260
approval_policy = "untrusted"
```

若您希望在指令失敗時收到提示，請使用 "on-failure"：

```toml
# 若在沙盒中執行指令失敗，Codex 會請求權限，以便在沙盒外重試。
approval_policy = "on-failure"
```

若您希望讓模型持續執行，直到判斷需要升級權限才請示，請使用 "on-request"：

```toml
# 由模型決定何時升級權限
approval_policy = "on-request"
```

或者，您也可以讓模型一路執行到結束，且永不請求以升級權限執行指令：

```toml
# 永不提示使用者：若指令失敗，Codex 會自動嘗試其他作法。
# 注意：`exec` 子指令一律使用此模式。
approval_policy = "never"
```

## profiles

_profile_ 是一組可一起設定的組態值。您可在 `config.toml` 定義多個 profile，並在執行時以 `--profile` 指定要使用的那一個。

以下為定義多個 profile 的 `config.toml` 範例：

```toml
model = "o3"
approval_policy = "untrusted"
disable_response_storage = false

# 設定 `profile` 等同於在命令列指定 `--profile o3`，
# 但仍可用 `--profile` 覆寫此值。
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

使用者可在多個層級指定組態值。優先序如下：

1. 自訂命令列參數，例如 `--model o3`
2. 來自某個 profile（透過 CLI 的 `--profile` 指定，或直接寫在組態中）
3. `config.toml` 檔案中的條目，例如 `model = "o3"`
4. Codex CLI 內建預設值（例如預設為 `gpt-5`）

## model_reasoning_effort

若所選模型支援 reasoning（例如：`o3`、`o4-mini`、`codex-*`、`gpt-5`），當使用 Responses API 時，預設會啟用 reasoning。根據［<a href="https://platform.openai.com/docs/guides/reasoning?api-mode=responses#get-started-with-reasoning">OpenAI 平台文件</a>］，此值可設定為：

- `"minimal"`
- `"low"`
- `"medium"` (default)
- `"high"`

注意：若要將 reasoning 降到最少，請選擇 `"minimal"`。

## model_reasoning_summary

若模型名稱以 `"o"`（如 `"o3"` 或 `"o4-mini"`）或 `"codex"` 開頭，使用 Responses API 時預設會啟用 reasoning。根據［<a href="https://platform.openai.com/docs/guides/reasoning?api-mode=responses#reasoning-summaries">OpenAI 平台文件</a>］，此值可設定為：

- `"auto"` (default)
- `"concise"`
- `"detailed"`

若要停用 reasoning 摘要，請在組態中將 `model_reasoning_summary` 設為 `"none"`：

```toml
model_reasoning_summary = "none"  # disable reasoning summaries
```

## model_verbosity

在使用 Responses API 時，控制 GPT‑5 系列模型的輸出長度/細節。可用值：

- `"low"`
- `"medium"` (default when omitted)
- `"high"`

設定後，Codex 會在請求 payload 中加入帶有此等級的 `text` 物件，例如：`"text": { "verbosity": "low" }`。

範例：

```toml
model = "gpt-5"
model_verbosity = "low"
```

注意：此設定僅適用於使用 Responses API 的供應商。使用 Chat Completions 的供應商不受影響。

## model_supports_reasoning_summaries

預設情況下，只有在確認支援 reasoning 的 OpenAI 模型請求上才會設定 `reasoning`。若要強制對目前模型的請求也設定 `reasoning`，可在 `config.toml` 中加入：

```toml
model_supports_reasoning_summaries = true
```

## sandbox_mode

Codex 會在作業系統層級的沙盒中執行模型產生的 shell 指令。

在多數情況下，您可用單一選項決定行為：

```toml
# 等同 `--sandbox read-only`
sandbox_mode = "read-only"
```

預設為 `read-only`，表示指令可讀取磁碟上的任何檔案，但嘗試寫入檔案或存取網路會被封鎖。

較寬鬆的策略是 `workspace-write`。指定後，Codex 工作的目前工作目錄會允許寫入（macOS 上的 `$TMPDIR` 亦同）。CLI 預設使用啟動所在的目錄作為 `cwd`，可透過 `--cwd/-C` 覆寫。

在 macOS（以及即將支援的 Linux）上，若可寫入的根目錄（包含 `cwd`）其下一層含有 `.git/` 資料夾，則會將 `.git/` 設為唯讀，而其餘 Git 內容可寫入。這表示像 `git commit` 之類需要寫入 `.git/` 的指令預設會失敗，並需要 Codex 請求權限。

```toml
# 等同 `--sandbox workspace-write`
sandbox_mode = "workspace-write"

# 僅在 `sandbox = "workspace-write"` 時適用的額外設定。
[sandbox_workspace_write]
# 預設情況下，Codex 會話的 cwd、$TMPDIR（若存在）與 /tmp（若存在）允許寫入。
# 將下列選項設為 `true` 可覆寫此預設。
exclude_tmpdir_env_var = false
exclude_slash_tmp = false

# 於 $TMPDIR 與 /tmp 之外可選擇加入「額外」可寫入根目錄。
writable_roots = ["/Users/YOU/.pyenv/shims"]

# 允許在沙盒中執行的指令可對外發送網路請求。預設為停用。
network_access = false
```

若要完全停用沙盒，請設定 `danger-full-access`：

```toml
# 等同 `--sandbox danger-full-access`
sandbox_mode = "danger-full-access"
```

若 Codex 於自身已提供沙盒的環境執行（例如 Docker 容器），可以合理地使用此選項而無需再加上額外沙盒。

此外，若您嘗試在不支援 Codex 原生沙盒機制的環境中使用（例如較舊的 Linux 核心或 Windows），也可能需要此選項。

## 核准預設組合

Codex 提供三種主要的核准預設組合：

- Read Only：Codex 可讀取檔案與回答問題；編輯、執行指令與網路存取需要核准。
- Auto：Codex 可在工作區內讀取、編輯與執行指令而不需核准；在工作區外或需網路存取時會請求核准。
- Full Access：不提示即擁有完整磁碟與網路存取權限；風險極高。

您可在命令列透過 `--ask-for-approval` 與 `--sandbox` 進一步自訂 Codex 的行為。

## mcp_servers

定義 Codex 可查詢以使用工具的 MCP 伺服器清單。目前僅支援以程式啟動並透過 stdio 通訊的伺服器。若伺服器使用 SSE 傳輸，請考慮使用類似 [mcp-proxy](https://github.com/sparfenyuk/mcp-proxy) 的轉接器。

**注意：**Codex 可能會快取 MCP 伺服器提供的工具與資源清單，以便在啟動時無需啟動所有伺服器就能將資訊放入脈絡中。此設計可透過延遲載入 MCP 伺服器來節省資源。

此設定與 Claude、Cursor 在其 JSON 組態中定義 `mcpServers` 的方式相當，但 Codex 使用 TOML，因此格式略有不同。例如，以下 JSON 組態：

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

在 `~/.codex/config.toml` 中應寫為：

```toml
# 重要：頂層 key 為 `mcp_servers` 而非 `mcpServers`。
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

## disable_response_storage

目前，帳戶設定為 Zero Data Retention（ZDR）的客戶必須將 `disable_response_storage` 設為 `true`，讓 Codex 採用符合 ZDR 的 Responses API 替代方案：

```toml
disable_response_storage = true
```

## shell_environment_policy

Codex 會產生子行程（例如執行助理建議的 `local_shell` 工具呼叫時）。預設會將「您的完整環境」傳遞給這些子行程。您可透過 `config.toml` 的 **`shell_environment_policy`** 區塊來調整此行為：

```toml
[shell_environment_policy]
# inherit 可為 "all"（預設）、"core" 或 "none"
inherit = "core"
# 設為 true 以「略過」`"*KEY*"` 與 `"*TOKEN*"` 的預設過濾
ignore_default_excludes = false
# 排除樣式（不分大小寫的 glob）
exclude = ["AWS_*", "AZURE_*"]
# 強制設定／覆寫值
set = { CI = "1" }
# 若提供此項，則「只」保留符合任一樣式的變數
include_only = ["PATH", "HOME"]
```

| 欄位                       | 型別                       | 預設值  | 說明                                                                                                                                             |
| ------------------------- | -------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `inherit`                 | string                     | `all`   | 環境的起始模板：<br>`all`（完整複製父層環境）、`core`（僅 `HOME`、`PATH`、`USER` 等）、或 `none`（空白開始）。                               |
| `ignore_default_excludes` | boolean                    | `false` | 當為 `false` 時，在其他規則執行前，Codex 會先移除名稱包含 `KEY`、`SECRET` 或 `TOKEN` 的變數（不分大小寫）。                                   |
| `exclude`                 | array<string>              | `[]`    | 預設過濾後要再排除的樣式（不分大小寫的 glob）。<br>例如：`"AWS_*"`、`"AZURE_*"`。                                                                |
| `set`                     | table<string,string>       | `{}`    | 明確指定要覆寫或新增的鍵值，優先於繼承值。                                                                                                     |
| `include_only`            | array<string>              | `[]`    | 若非空，表示白名單樣式；只有符合其中一個樣式的變數會在最後被保留。（通常搭配 `inherit = "all"` 使用。）                                        |

樣式採用「glob」而非完整正規表示式：`*` 代表任意多個字元，`?` 代表單一字元，也支援字元類別如 `[A-Z]`/`[^0-9]`。比對一律「不分大小寫」。此語法在程式碼中以 `EnvironmentVariablePattern` 記載（見 `core/src/config_types.rs`）。

若您只需要乾淨的環境，並加入少量自訂項目，可如下設定：

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
```

目前在假定網路停用的情況下，環境中也會加入 `CODEX_SANDBOX_NETWORK_DISABLED=1`。此行為不可設定。

## notify

指定一個程式，用於接收 Codex 產生之事件的通知。該程式會以字串形式接收 JSON 參數，例如：

```json
{
  "type": "agent-turn-complete",
  "turn-id": "12345",
  "input-messages": ["Rename `foo` to `bar` and update the callsites."],
  "last-assistant-message": "Rename complete and verified `cargo build` succeeds."
}
```

`"type"` 屬性一定會存在。目前僅支援 `"agent-turn-complete"` 這一種通知類型。

以下是一個 Python 範例腳本，會解析 JSON，並決定是否在 macOS 使用 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 顯示桌面推播通知：

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

若要讓 Codex 使用此腳本發送通知，請在 `~/.codex/config.toml` 的 `notify` 設定中填入您電腦上 `notify.py` 的正確路徑：

```toml
notify = ["python3", "/Users/mbolin/.codex/notify.py"]
```

## history

預設情況下，Codex CLI 會將傳送給模型的訊息記錄在 `$CODEX_HOME/history.jsonl`。在 UNIX 上，該檔案權限為 `o600`，因此僅擁有者可讀寫。

若要停用此行為，請在 `[history]` 中設定如下：

```toml
[history]
persistence = "none"  # "save-all" is the default value
```

## file_opener

指定在模型輸出中的檔案引用要使用的編輯器/URI 方案。若設定此項，輸出中的檔案引用會以指定的 URI 方案加上超連結，您可在終端機中使用 Ctrl/Cmd‑Click 開啟。

例如，若輸出包含 `【F:/home/user/project/main.py†L42-L50】`，則會改寫為連結 `vscode://file/home/user/project/main.py:42`。

注意：這不是一般的編輯器設定（例如 `$EDITOR`），因為它只接受固定的一組值：

- `"vscode"` (default)
- `"vscode-insiders"`
- `"windsurf"`
- `"cursor"`
- `"none"` to explicitly disable this feature

目前預設為 `"vscode"`，但 Codex 並不會驗證是否已安裝 VS Code。未來 `file_opener` 可能會改為預設 `"none"` 或其他值。

## hide_agent_reasoning

Codex 會不定期發出「reasoning」事件，以呈現模型在產生最終答案前的內部「思考」。有些使用者可能會覺得這些事件干擾，特別是在 CI 記錄或精簡的終端輸出中。

將 `hide_agent_reasoning` 設為 `true` 可在 TUI 與無頭的 `exec` 子指令中同時隱藏這些事件：

```toml
hide_agent_reasoning = true   # defaults to false
```

## show_raw_agent_reasoning

在可用時顯示模型的原始推理過程（raw reasoning content）。

注意事項：

- 僅當所選模型/供應商實際提供 raw reasoning content 時才會生效。許多模型不支援；不支援時，這個選項不會有可見效果。
- 原始推理內容可能包含中間思考或敏感脈絡。僅在您的流程可接受時啟用。

範例：

```toml
show_raw_agent_reasoning = true  # defaults to false
```

## model_context_window

模型的脈絡視窗大小（token 數）。

一般而言，Codex 知道常見 OpenAI 模型的脈絡視窗大小，但若您在舊版 Codex CLI 搭配新模型時，可透過 `model_context_window` 告訴 Codex 要用多少值來判斷對話剩餘脈絡。

## model_max_output_tokens

與 `model_context_window` 類似，但此為模型輸出 token 的上限。

## project_doc_max_bytes

在會話第一輪的指示中，從 `AGENTS.md` 讀取的最大位元組數。預設為 32 KiB。

## tui

TUI 專屬的選項。

```toml
[tui]
# 後續將補充
```

## 設定參考

| Key | 型別／值 | 說明 |
| --- | --- | --- |
| `model` | string | 要使用的模型（例如 `gpt-5`）。 |
| `model_provider` | string | 來自 `model_providers` 的供應商 id（預設：`openai`）。 |
| `model_context_window` | number | 脈絡視窗的 token 數。 |
| `model_max_output_tokens` | number | 輸出 token 上限。 |
| `approval_policy` | `untrusted` \| `on-failure` \| `on-request` \| `never` | 何時提示核准。 |
| `sandbox_mode` | `read-only` \| `workspace-write` \| `danger-full-access` | OS 沙盒策略。 |
| `sandbox_workspace_write.writable_roots` | array<string> | 在 workspace‑write 模式中的額外可寫入根目錄。 |
| `sandbox_workspace_write.network_access` | boolean | 在 workspace‑write 中是否允許網路（預設：false）。 |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean | 從可寫入根目錄排除 `$TMPDIR`（預設：false）。 |
| `sandbox_workspace_write.exclude_slash_tmp` | boolean | 從可寫入根目錄排除 `/tmp`（預設：false）。 |
| `disable_response_storage` | boolean | ZDR 組織必須。 |
| `notify` | array<string> | 外部通知程式。 |
| `instructions` | string | 目前忽略；請用 `experimental_instructions_file` 或 `AGENTS.md`。 |
| `mcp_servers.<id>.command` | string | MCP 伺服器啟動指令。 |
| `mcp_servers.<id>.args` | array<string> | MCP 伺服器引數。 |
| `mcp_servers.<id>.env` | map<string,string> | MCP 伺服器環境變數。 |
| `model_providers.<id>.name` | string | 顯示名稱。 |
| `model_providers.<id>.base_url` | string | API 基底 URL。 |
| `model_providers.<id>.env_key` | string | API 金鑰的環境變數。 |
| `model_providers.<id>.wire_api` | `chat` \| `responses` | 使用的通訊協定（預設：`chat`）。 |
| `model_providers.<id>.query_params` | map<string,string> | 額外查詢參數（如 Azure 的 `api-version`）。 |
| `model_providers.<id>.http_headers` | map<string,string> | 額外靜態標頭。 |
| `model_providers.<id>.env_http_headers` | map<string,string> | 從環境變數帶入的標頭。 |
| `model_providers.<id>.request_max_retries` | number | 供應商層級的 HTTP 重試次數（預設：4）。 |
| `model_providers.<id>.stream_max_retries` | number | SSE 串流重試次數（預設：5）。 |
| `model_providers.<id>.stream_idle_timeout_ms` | number | SSE 閒置逾時（毫秒）（預設：300000）。 |
| `project_doc_max_bytes` | number | 從 `AGENTS.md` 讀取的最大位元組數。 |
| `profile` | string | 啟用中的 profile 名稱。 |
| `profiles.<name>.*` | various | 以 profile 範圍覆寫相同鍵。 |
| `history.persistence` | `save-all` \| `none` | 歷史檔持久化（預設：`save-all`）。 |
| `history.max_bytes` | number | 目前忽略（未強制）。 |
| `file_opener` | `vscode` \| `vscode-insiders` \| `windsurf` \| `cursor` \| `none` | 可點擊引文的 URI 方案（預設：`vscode`）。 |
| `tui` | table | TUI 專屬選項（保留）。 |
| `hide_agent_reasoning` | boolean | 隱藏模型 reasoning 事件。 |
| `show_raw_agent_reasoning` | boolean | 顯示原始 reasoning（若可用）。 |
| `model_reasoning_effort` | `minimal` \| `low` \| `medium` \| `high` | Responses API 的 reasoning 強度。 |
| `model_reasoning_summary` | `auto` \| `concise` \| `detailed` \| `none` | reasoning 摘要。 |
| `model_verbosity` | `low` \| `medium` \| `high` | GPT‑5 文字詳略（Responses API）。 |
| `model_supports_reasoning_summaries` | boolean | 強制啟用 reasoning 摘要。 |
| `chatgpt_base_url` | string | ChatGPT 驗證流程的基底 URL。 |
| `experimental_resume` | string (path) | Resume JSONL 路徑（內部/實驗）。 |
| `experimental_instructions_file` | string (path) | 取代內建指示（實驗）。 |
| `experimental_use_exec_command_tool` | boolean | 使用實驗性的 exec 指令工具。 |
| `use_experimental_reasoning_summary` | boolean | 對推理鏈使用實驗性摘要。 |
| `responses_originator_header_internal_override` | string | 覆寫 `originator` 標頭值。 |
| `projects.<path>.trust_level` | string | 標記專案/工作樹為信任（目前僅辨識 `"trusted"`）。 |
| `preferred_auth_method` | `chatgpt` \| `apikey` | 預設驗證方式（預設：`chatgpt`）。 |
| `tools.web_search` | boolean | 啟用網頁搜尋工具（別名：`web_search_request`）（預設：false）。 |
