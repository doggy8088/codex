## 零資料保留 (ZDR) 用法

Codex CLI **確實**支援已啟用 [零資料保留 (ZDR)](https://platform.openai.com/docs/guides/your-data#zero-data-retention) 的 OpenAI 組織。如果您的 OpenAI 組織已啟用零資料保留，但您仍然遇到諸如以下的錯誤：

```
OpenAI rejected the request. Error details: Status: 400, Code: unsupported_parameter, Type: invalid_request_error, Message: 400 Previous response cannot be used for this organization due to Zero Data Retention.
```

請確保您在執行 `codex` 時加上 `--config disable_response_storage=true`，或將此行新增至 `~/.codex/config.toml`，以避免每次都需指定命令列選項：

```toml
disable_response_storage = true
```

詳情請參閱[關於 `disable_response_storage` 的設定文件](./config.md#disable_response_storage)。