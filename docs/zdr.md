## 零資料保留 (ZDR) 使用說明

Codex CLI **確實** 支援啟用了[零資料保留 (ZDR)](https://platform.openai.com/docs/guides/your-data#zero-data-retention) 的 OpenAI 組織。如果您的 OpenAI 組織已啟用零資料保留，但仍遇到如下錯誤：

```
OpenAI rejected the request. Error details: Status: 400, Code: unsupported_parameter, Type: invalid_request_error, Message: 400 Previous response cannot be used for this organization due to Zero Data Retention.
```

請確保您使用 `--config disable_response_storage=true` 執行 `codex`，或將此行加入 `~/.codex/config.toml` 以避免每次都需要指定命令列選項：

```toml
disable_response_storage = true
```

詳細資訊請參閱[`disable_response_storage` 的設定文件](./config.md#disable_response_storage)。