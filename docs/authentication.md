# 驗證

## 依使用量計費的替代方案：使用 OpenAI API 金鑰

若您偏好隨用隨付，仍可透過設定 OpenAI API 金鑰為環境變數進行驗證：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

此金鑰至少需要具備對 Responses API 的寫入權限。

## 從 API 金鑰移轉到 ChatGPT 登入

如果您先前以 API 金鑰搭配依使用量計費使用 Codex CLI，現在想改為使用 ChatGPT 方案，請依下列步驟操作：

1. 升級 CLI 並確認 `codex --version` 為 `0.20.0` 或以上
2. 刪除 `~/.codex/auth.json`（Windows：`C:\\Users\\USERNAME\\.codex\\auth.json`）
3. 再次執行 `codex login`

## 強制使用特定驗證方式（進階）

當兩種方式都可用時，您可以明確選擇 Codex 優先使用哪一種驗證方式。

- 若要一律使用 API 金鑰（即使存在 ChatGPT 驗證），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "apikey"
```

或在 CLI 以臨時覆寫：

```bash
codex --config preferred_auth_method="apikey"
```

- 若要偏好使用 ChatGPT 驗證（預設），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "chatgpt"
```

注意事項：

- 當 `preferred_auth_method = "apikey"` 且可取得 API 金鑰時，會略過登入畫面。
- 當 `preferred_auth_method = "chatgpt"`（預設）時，若可使用 ChatGPT 驗證，Codex 會優先使用；若僅有 API 金鑰，則改用 API 金鑰。某些帳戶類型也可能需要 API 金鑰模式。
- 想在會話期間檢視目前使用的驗證方式，可在 TUI 中使用 `/status` 指令。

## 在「無頭」機器上連線

目前登入流程會在 `localhost:1455` 啟動一個伺服器。若您使用的是「無頭」環境，例如 Docker 容器或透過 `ssh` 連線到遠端機器，在本機瀏覽器開啟 `localhost:1455` 並不會自動連線到無頭機器上的網站伺服器，因此請使用以下任一替代作法：

### 在本機完成驗證後，將認證檔複製到「無頭」機器

最簡單的方式是在本機執行 `codex login`，確保您可以在瀏覽器中存取 `localhost:1455`。完成驗證後，`auth.json` 會出現在 `$CODEX_HOME/auth.json`（在 Mac/Linux，`$CODEX_HOME` 預設為 `~/.codex`；在 Windows 則為 `%USERPROFILE%\\.codex`）。

由於 `auth.json` 不綁定特定主機，一旦您在本機完成驗證流程，便可將 `$CODEX_HOME/auth.json` 複製到無頭機器上，之後該機器上的 `codex` 應可直接使用。若要複製檔案到 Docker 容器，可使用：

```shell
# substitute MY_CONTAINER with the name or id of your Docker container:
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

若您是以 `ssh` 連線到遠端機器，通常會使用［<a href="https://en.wikipedia.org/wiki/Secure_copy_protocol">`scp`</a>］：

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

或嘗試以下單行指令：

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### 透過 VPS 或遠端連線

如果您在遠端機器（VPS/伺服器）上執行 Codex 而無法使用本機瀏覽器，登入協助工具會在遠端主機的 `localhost:1455` 啟動伺服器。要在本機瀏覽器完成登入，請在開始登入流程前，先將該連接埠轉送到您的電腦：

```bash
# From your local machine
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

接著在該 SSH 連線中執行 `codex` 並選擇「Sign in with ChatGPT」。出現提示時，請在本機瀏覽器開啟列印出來的 URL（會是 `http://localhost:1455/...`）。流量會透過通道轉送至遠端伺服器。
