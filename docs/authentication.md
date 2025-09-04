# 身份驗證

## 即用即付的替代方案：使用 OpenAI API 金鑰

如果您偏好即用即付，仍然可以透過將您的 OpenAI API 金鑰設定為環境變數來進行驗證：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

此金鑰至少必須具備 Responses API 的寫入權限。

## 從 API 金鑰遷移至 ChatGPT 登入

如果您之前曾透過 API 金鑰使用 Codex CLI 的即用即付模式，且想要改用您的 ChatGPT 方案，請遵循以下步驟：

1. 更新 CLI 並確保 `codex --version` 為 `0.20.0` 或更新版本
2. 刪除 `~/.codex/auth.json` （在 Windows 上為：`C:\\Users\\USERNAME\\.codex\\auth.json`）
3. 再次執行 `codex login`

## 強制指定特定驗證方式（進階）

當兩種驗證方式皆可用時，您可以明確選擇 Codex 應優先使用哪一種。

- 若要一律使用您的 API 金鑰（即使已存在 ChatGPT 驗證），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "apikey"
```

或透過 CLI 臨時覆寫：

```bash
codex --config preferred_auth_method="apikey"
```

- 若要優先使用 ChatGPT 驗證（預設），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "chatgpt"
```

注意事項：

- 當 `preferred_auth_method = "apikey"` 且有可用的 API 金鑰時，將會跳過登入畫面。
- 當 `preferred_auth_method = "chatgpt"`（預設）時，若 ChatGPT 驗證存在，Codex 會優先使用；若僅有 API 金鑰存在，則會使用 API 金鑰。某些帳戶類型可能也需要 API 金鑰模式。
- 若要檢查工作階段期間正在使用哪種驗證方式，請在 TUI 中使用 `/status` 指令。

## 在「無頭機器」上連線

目前，登入流程需要在 `localhost:1455` 上執行一個伺服器。如果您在「無頭」伺服器上（例如 Docker 容器，或透過 `ssh` 連線到遠端機器），在您本機的瀏覽器中載入 `localhost:1455` 並不會自動連線到在_無頭_機器上執行的網頁伺服器，因此您必須使用下列其中一種解決方案：

### 在本機進行驗證並將憑證複製到「無頭」機器

最簡單的解決方案可能是在您的本機上執行 `codex login` 流程，讓您的網頁瀏覽器可以存取 `localhost:1455`。當您完成驗證流程後，一個 `auth.json` 檔案應該會出現在 `$CODEX_HOME/auth.json`（在 Mac/Linux 上，`$CODEX_HOME` 預設為 `~/.codex`，而在 Windows 上則預設為 `%USERPROFILE%\\.codex`）。

由於 `auth.json` 檔案並未綁定特定主機，一旦您在本機完成驗證流程，就可以將 `$CODEX_HOME/auth.json` 檔案複製到無頭機器上，接著 `codex` 應該就能在那台機器上「直接運作」。請注意，若要將檔案複製到 Docker 容器，您可以執行：

```shell
# substitute MY_CONTAINER with the name or id of your Docker container:
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

然而，如果您是 `ssh` 進入遠端機器，您可能會想使用 [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol)：

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

或嘗試這個單行指令：

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### 透過 VPS 或遠端連線

如果您在遠端機器 (VPS/伺服器) 上執行 Codex 且沒有本機瀏覽器，登入輔助程式會在遠端主機的 `localhost:1455` 上啟動一個伺服器。為了在您的本機瀏覽器中完成登入，請在開始登入流程前，將該端口轉發到您的機器：

```bash
# From your local machine
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

然後，在該 SSH 工作階段中，執行 `codex` 並選擇「使用 ChatGPT 登入」。當出現提示時，在您的本機瀏覽器中開啟印出的 URL (它將是 `http://localhost:1455/...`)。流量將會透過隧道傳輸到遠端伺服器。
