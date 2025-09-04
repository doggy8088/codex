## 貢獻

此專案正在積極開發中，程式碼可能會有相當大的變化。

**目前，我們只計劃優先審查外部對錯誤或安全修復的貢獻。**

如果您想新增新功能或改變現有功能的行為，請在花時間開發之前先開啟 issue 提案該功能，並獲得 OpenAI 團隊成員的核准。

**未經此流程的新貢獻可能會被關閉**，如果它們與我們目前的路線圖不一致或與其他優先事項/即將推出的功能衝突。

### 開發工作流程

- 從 `main` 建立一個 _主題分支_ - 例如 `feat/interactive-prompt`。
- 保持您的變更聚焦。多個不相關的修復應該作為單獨的 PR 開啟。
- 遵循上述[開發設定](#development-workflow)指示，確保您的變更沒有 lint 警告和測試失敗。

### 撰寫高影響力的程式碼變更

1. **從 issue 開始。** 開啟新的 issue 或在現有討論上留言，以便我們在撰寫程式碼之前就解決方案達成一致。
2. **新增或更新測試。** 每個新功能或錯誤修復都應該附帶測試覆蓋，在您的變更前失敗，在變更後通過。不需要 100% 覆蓋率，但要追求有意義的斷言。
3. **記錄行為。** 如果您的變更影響使用者面向的行為，請更新 README、內聯說明（`codex --help`）或相關範例專案。
4. **保持提交原子性。** 每個提交都應該能編譯且測試應該通過。這使審查和潛在的回滾更容易。

### 開啟 pull request

- 填寫 PR 模板（或包含類似資訊）- **什麼？為什麼？如何？**
- 在本機執行 **所有** 檢查（`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`）。可在本機捕獲的 CI 失敗會拖慢流程。
- 確保您的分支與 `main` 同步，並且您已解決合併衝突。
- 只有當您認為 PR 處於可合併狀態時，才將其標記為 **Ready for review**。

### 審查流程

1. 一位維護者將被指派為主要審查者。
2. 如果您的 PR 新增了先前未討論和核准的新功能，我們可能選擇關閉您的 PR（參見[貢獻](#contributing)）。
3. 我們可能要求變更 - 請不要將此視為針對個人。我們重視工作，但我們也重視一致性和長期可維護性。
5. 當共識認為 PR 達到標準時，維護者將執行 squash-and-merge。

### 社群價值觀

- **保持友善和包容。** 尊重他人；我們遵循[貢獻者公約](https://www.contributor-covenant.org/)。
- **假設善意。** 書面溝通很困難 - 傾向於寬容。
- **教學與學習。** 如果您發現令人困惑的地方，請開啟 issue 或 PR 提出改進建議。

### 獲得協助

如果您在設定專案時遇到問題、想要對想法獲得回饋，或只是想說聲 _嗨_ - 請開啟討論或參與相關的 issue。我們很樂意協助。

我們一起可以讓 Codex CLI 成為一個令人驚嘆的工具。**開心駭客！** :rocket:

### 貢獻者授權協議 (CLA)

所有貢獻者 **必須** 接受 CLA。流程很輕量：

1. 開啟您的 pull request。
2. 貼上以下評論（或如果您之前已簽署則回覆 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 機器人會在儲存庫中記錄您的簽名並將狀態檢查標記為通過。

不需要特殊的 Git 命令、電子郵件附件或提交頁腳。

#### 快速修復

| 情境              | 命令                                             |
| ----------------- | ------------------------------------------------ |
| 修改最後一次提交  | `git commit --amend -s --no-edit && git push -f` |

**DCO 檢查** 會阻止合併，直到 PR 中的每個提交都帶有頁腳（使用 squash 時只是其中一個）。

### 發布 `codex`

_僅限管理員。_

確保您在 `main` 上且沒有本機變更。然後執行：

```shell
VERSION=0.2.0  # 也可以是 0.2.0-alpha.1 或任何有效的 Rust 版本。
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這將在 `main` 頂部進行本機提交，在 `codex-rs/Cargo.toml` 中將 `version` 設定為 `$VERSION`（注意在 `main` 上，我們將版本保留為 `version = "0.0.0"`）。

這將使用標籤 `rust-v${VERSION}` 推送提交，進而啟動[發布工作流程](../.github/workflows/rust-release.yml)。這將建立名為 `$VERSION` 的新 GitHub Release。

如果生成的 GitHub Release 看起來不錯，請取消勾選 **pre-release** 框以使其成為最新版本。

建立 PR 以更新 Homebrew 上的 [`Formula/c/codex.rb`](https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb)。

### 安全與負責任的 AI

您是否發現了漏洞或對模型輸出有疑慮？請發送電子郵件至 **security@openai.com**，我們會迅速回應。