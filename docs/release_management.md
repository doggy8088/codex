# 發布管理

目前，我們在三個地方提供 Codex 二進位檔案：

- GitHub Releases https://github.com/openai/codex/releases/
- npm 上的 `@openai/codex`：https://www.npmjs.com/package/@openai/codex
- Homebrew 上的 `codex`：https://formulae.brew.sh/formula/codex

# 切出版本

目前，為下一個版本選擇版本號是一個手動過程。一般來說，只需前往 https://github.com/openai/codex/releases/latest 查看最新版本，並將次版本號增加 `1`，所以如果目前版本是 `0.20.0`，下一個版本就應該是 `0.21.0`。

假設您想要發布 `0.21.0`，首先您需要執行：

```shell
VERSION=0.21.0
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這將啟動一個 GitHub Action 來建置版本，因此請前往 https://github.com/openai/codex/actions/workflows/rust-release.yml 尋找對應的工作流程。（注意：我們應該用 `gh` 自動化找到工作流程 URL。）

當工作流程完成時，GitHub Release 就「完成」了，但您仍需要考慮 npm 和 Homebrew。

## 發布到 npm

GitHub Release 完成後，您可以發布到 npm。注意 GitHub Release 包含 npm 的適當構件（即 `npm pack` 的輸出），應該命名為 `codex-npm-VERSION.tgz`。要發布到 npm，請執行：

```
VERSION=0.21.0
./scripts/publish_to_npm.py "$VERSION"
```

請注意，您必須有權限發布到 https://www.npmjs.com/package/@openai/codex 才能成功。

## 發布到 Homebrew

對於 Homebrew，我們已經正確設定了他們的自動化系統，所以大約每隔幾小時它會檢查我們的 GitHub 儲存庫以查看是否有新版本。當它發現新版本時，會提出 PR 來建立等效的 Homebrew 版本，這需要在各種版本的 macOS 上從原始碼建置 Codex CLI。

不可避免地，您只需要定期重新整理此頁面，查看版本是否已被他們的自動化系統取得：

https://github.com/Homebrew/homebrew-core/pulls?q=%3Apr+codex

一旦所有建置完成，Homebrew 管理員必須核准 PR。同樣，整個過程需要數小時，我們對此沒有完全控制權，但似乎運作得很好。

作為參考，我們的 Homebrew formula 位於：

https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb