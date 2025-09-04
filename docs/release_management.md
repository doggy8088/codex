# 版本發布管理

目前，我們在三個地方提供 Codex 二進位檔：

- GitHub Releases https://github.com/openai/codex/releases/
- `@openai/codex` on npm: https://www.npmjs.com/package/@openai/codex
- `codex` on Homebrew: https://formulae.brew.sh/formula/codex

# 發布版本

目前，為下一個版本選擇版本號是一個手動過程。一般來說，只需前往 https://github.com/openai/codex/releases/latest 查看最新版本，並將次要版本號增加 `1`，所以如果目前版本是 `0.20.0`，那麼下一個版本就應該是 `0.21.0`。

假設您要發布 `0.21.0`，首先您需要執行：

```shell
VERSION=0.21.0
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這將會觸發一個 GitHub Action 來建構發布版本，所以請前往 https://github.com/openai/codex/actions/workflows/rust-release.yml 尋找對應的工作流程。（注意：我們應該使用 `gh` 來自動化尋找工作流程 URL 的過程。）

當工作流程完成後，GitHub Release 就「完成」了，但您仍然需要考慮 npm 和 Homebrew。

## 發布至 npm

在 GitHub Release 完成後，您就可以發布到 npm。請注意，GitHub Release 包含適用於 npm 的成品（即 `npm pack` 的輸出），其名稱應為 `codex-npm-VERSION.tgz`。要發布到 npm，請執行：

```
VERSION=0.21.0
./scripts/publish_to_npm.py "$VERSION"
```

請注意，您必須擁有發布到 https://www.npmjs.com/package/@openai/codex 的權限才能成功。

## 發布至 Homebrew

對於 Homebrew，我們已經與他們的自動化系統正確設定，因此它會每隔數小時檢查我們的 GitHub repo 是否有新版本。當它找到新版本時，就會提出一個 PR 來建立對應的 Homebrew 版本，這需要在各種 macOS 版本上從原始碼建構 Codex CLI。

無可避免地，您必須定期重新整理此頁面，以查看發布是否已被其自動化系統接收：

https://github.com/Homebrew/homebrew-core/pulls?q=%3Apr+codex

一旦所有內容都建構完成，Homebrew 管理員就必須核准該 PR。同樣地，整個過程需要數小時，我們無法完全控制，但它似乎運作得相當不錯。

作為參考，我們的 Homebrew formula 位於：

https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb
