# 版本發佈管理

目前，我們在三個地方提供 Codex 的可執行檔：

- GitHub Releases：https://github.com/openai/codex/releases/
- npm 的 `@openai/codex`：https://www.npmjs.com/package/@openai/codex
- Homebrew 的 `codex`：https://formulae.brew.sh/formula/codex

# 建立新版本

目前，下一個版本號是手動決定。一般來說，前往 https://github.com/openai/codex/releases/latest 查看最新版本，將次版號加 `1` 即可：若目前為 `0.20.0`，下一版應為 `0.21.0`。

假設要發佈 `0.21.0`，請先執行：

```shell
VERSION=0.21.0
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這會啟動 GitHub Action 建置釋出。請前往 https://github.com/openai/codex/actions/workflows/rust-release.yml 找到對應的工作流程。（備註：我們應該用 `gh` 自動化找出該 URL。）

工作流程完成後，GitHub Release 即告完成，但仍需處理 npm 與 Homebrew。

## 發佈到 npm

完成 GitHub Release 後，即可發佈到 npm。請注意 GitHub Release 會包含給 npm 使用的成品（`npm pack` 的輸出），檔名為 `codex-npm-VERSION.tgz`。若要發佈到 npm，請執行：

```
VERSION=0.21.0
./scripts/publish_to_npm.py "$VERSION"
```

請務必確認您具備 https://www.npmjs.com/package/@openai/codex 的發佈權限。

## 發佈到 Homebrew

針對 Homebrew，我們已與其自動化系統串接。每隔幾小時會檢查我們的 GitHub 儲存庫是否有新版本。一旦發現，便會發 PR 建立對應的 Homebrew 版本，並在各 macOS 版本上從原始碼建置 Codex CLI。

實務上，您需要定期重新整理以下頁面，查看是否已被其自動化系統拾取：

https://github.com/Homebrew/homebrew-core/pulls?q=%3Apr+codex

所有建置完成後，尚需 Homebrew 管理員核准該 PR。整體流程需耗時數小時，我們無法完全掌控，但運作狀況普遍良好。

參考連結，我們的 Homebrew 配方位於：

https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb
