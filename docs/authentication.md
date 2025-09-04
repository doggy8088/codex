# 認證

## 基於使用量計費的替代方案：使用 OpenAI API 金鑰

如果您偏好按使用量付費，您仍可以透過將 OpenAI API 金鑰設定為環境變數來進行認證：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

此金鑰至少必須具有 Responses API 的寫入權限。

## 從 API 金鑰遷移到 ChatGPT 登入

如果您之前使用 Codex CLI 透過 API 金鑰進行基於使用量的計費，並想切換到使用您的 ChatGPT 方案，請遵循以下步驟：

1. 更新 CLI 並確保 `codex --version` 是 `0.20.0` 或更新版本
2. 刪除 `~/.codex/auth.json`（在 Windows 上：`C:\\Users\\USERNAME\\.codex\\auth.json`）
3. 再次執行 `codex login`

## 強制特定認證方法（進階）

當兩種認證方式都可用時，您可以明確選擇 Codex 應該偏好哪種認證方式。

- 要總是使用您的 API 金鑰（即使存在 ChatGPT 認證），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "apikey"
```

或透過 CLI 臨時覆蓋：

```bash
codex --config preferred_auth_method="apikey"
```

- 要偏好 ChatGPT 認證（預設），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "chatgpt"
```

注意事項：

- 當 `preferred_auth_method = "apikey"` 且 API 金鑰可用時，會跳過登入畫面。
- 當 `preferred_auth_method = "chatgpt"`（預設）時，如果存在 ChatGPT 認證，Codex 會偏好使用；如果僅存在 API 金鑰，則會使用 API 金鑰。某些帳戶類型可能也需要 API 金鑰模式。
- 要檢查在會話期間使用的認證方法，請在 TUI 中使用 `/status` 命令。

## 在「無頭」機器上連接

目前，登入流程需要在 `localhost:1455` 上執行伺服器。如果您在「無頭」伺服器上（如 Docker 容器或透過 `ssh` 連接到遠端機器），在本機瀏覽器中載入 `localhost:1455` 不會自動連接到在 _無頭_ 機器上執行的網頁伺服器，因此您必須使用以下其中一種變通方法：

### 在本機認證並將您的憑證複製到「無頭」機器

最簡單的解決方案可能是在您的本機上執行 `codex login` 流程，使 `localhost:1455` _可以_ 在您的網頁瀏覽器中存取。當您完成認證流程時，`auth.json` 檔案應該在 `$CODEX_HOME/auth.json` 位置可用（在 Mac/Linux 上，`$CODEX_HOME` 預設為 `~/.codex`，而在 Windows 上，預設為 `%USERPROFILE%\\.codex`）。

因為 `auth.json` 檔案不綁定到特定主機，一旦您在本機完成認證流程，您可以將 `$CODEX_HOME/auth.json` 檔案複製到無頭機器，然後 `codex` 在該機器上應該「就能運作」。注意要複製檔案到 Docker 容器，您可以執行：

```shell
# 將 MY_CONTAINER 替換為您 Docker 容器的名稱或 id：
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

而如果您透過 `ssh` 連接到遠端機器，您可能想要使用 [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol)：

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

或嘗試這個一行命令：

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### 透過 VPS 或遠端連接

如果您在沒有本機瀏覽器的遠端機器（VPS/伺服器）上執行 Codex，登入協助程式會在遠端主機的 `localhost:1455` 上啟動伺服器。要在您的本機瀏覽器中完成登入，請在開始登入流程之前將該連接埠轉發到您的機器：

```bash
# 從您的本機
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

然後，在該 SSH 會話中，執行 `codex` 並選擇「使用 ChatGPT 登入」。當提示時，在您的本機瀏覽器中開啟列印的 URL（它會是 `http://localhost:1455/...`）。流量會被隧道傳送到遠端伺服器。