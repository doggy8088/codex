## 零資料保留（ZDR）使用方式

Codex CLI **支援**啟用［<a href="https://platform.openai.com/docs/guides/your-data#zero-data-retention">Zero Data Retention（ZDR）</a>］的 OpenAI 組織。若您的組織已啟用 ZDR，但仍遇到如下錯誤：

```
OpenAI rejected the request. Error details: Status: 400, Code: unsupported_parameter, Type: invalid_request_error, Message: 400 Previous response cannot be used for this organization due to Zero Data Retention.
```

請確認您以 `--config disable_response_storage=true` 執行 `codex`，或將以下設定加入 `~/.codex/config.toml`，避免每次在命令列指定：

```toml
disable_response_storage = true
```

詳細說明請參閱［<a href="./config.md#disable_response_storage">`disable_response_storage` 組態文件</a>］。 
