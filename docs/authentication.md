# 身份驗證

## 用量計費替代方案：使用 OpenAI API 金鑰

如果您偏好按使用量付費，您仍然可以透過將 OpenAI API 金鑰設定為環境變數來進行身份驗證：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

此金鑰至少必須具有 Responses API 的寫入權限。

## 從 API 金鑰遷移到 ChatGPT 登入

如果您之前透過 API 金鑰使用 codex CLI 進行用量計費，並希望切換到使用您的 ChatGPT 方案使用 Codex，請按照以下步驟：

1. 更新 CLI 並確保 `codex --version` 是 `0.20.0` 或更高版本
2. 刪除 `~/.codex/auth.json`（在 Windows 上：`C:\\Users\\USERNAME\\.codex\\auth.json`）
3. 再次執行 `codex login`

## 強制使用特定身份驗證方法（進階）

當兩種方法都可用時，您可以明確選擇 codex 應該偏好哪種身份驗證。

- 要始終使用您的 API 金鑰（即使存在 ChatGPT 身份驗證），請設定：

```toml
# ~/.codex/config.toml
preferred_auth_method = "apikey"
```

或 override ad-hoc via CLI:

```bash
codex --config preferred_auth_method="apikey"
```

- 到 prefer ChatGPT auth (default), set:

```toml
# ~/.codex/config.toml
preferred_auth_method = "chatgpt"
```

Notes:

- When `preferred_auth_method = "apikey"` and an API key is available, the login screen is skipped.
- When `preferred_auth_method = "chatgpt"` (default), Codex prefers ChatGPT auth if present; if only an API key is present, it will use the API key. Certain account types may also require API-key mode.
- To check which auth method is being used during a session, use the `/status` command in the TUI.

## Connecting 在 a "Headless" Machine

Today, the login process entails running a server on `localhost:1455`. If you are on a "headless" server, such as a Docker container or are `ssh`'d into a remote machine, loading `localhost:1455` in the browser on your local machine will not automatically connect to the webserver running on the _headless_ machine, so you must use one of the following workarounds:

### Authenticate locally 和 copy 您的 credentials 到 the "headless" machine

The easiest solution is likely to run through the `codex login` process on your local machine such that `localhost:1455` _is_ accessible in your web browser. When you complete the authentication process, an `auth.json` file should be available at `$CODEX_HOME/auth.json` (on Mac/Linux, `$CODEX_HOME` defaults to `~/.codex` whereas on Windows, it defaults to `%USERPROFILE%\\.codex`).

Because the `auth.json` file is not tied to a specific host, once you complete the authentication flow locally, you can copy the `$CODEX_HOME/auth.json` file to the headless machine and then `codex` should "just work" on that machine. Note to copy a file to a Docker container, you can do:

```shell
# substitute MY_CONTAINER with the name or id of your Docker container:
CONTAINER_HOME=$(docker exec MY_CONTAINER printenv HOME)
docker exec MY_CONTAINER mkdir -p "$CONTAINER_HOME/.codex"
docker cp auth.json MY_CONTAINER:"$CONTAINER_HOME/.codex/auth.json"
```

whereas if you are `ssh`'d into a remote machine, you likely want to use [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol):

```shell
ssh user@remote 'mkdir -p ~/.codex'
scp ~/.codex/auth.json user@remote:~/.codex/auth.json
```

或 try 這個 one-liner:

```shell
ssh user@remote 'mkdir -p ~/.codex && cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### Connecting through VPS 或 remote

If you run Codex on a remote machine (VPS/server) without a local browser, the login helper starts a server on `localhost:1455` on the remote host. To complete login in your local browser, forward that port to your machine before starting the login flow:

```bash
# From your local machine
ssh -L 1455:localhost:1455 <user>@<remote-host>
```

Then, in that SSH session, run `codex` and select "Sign in with ChatGPT". When prompted, open the printed URL (it will be `http://localhost:1455/...`) in your local browser. The traffic will be tunneled to the remote server. 
