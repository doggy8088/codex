## 貢獻指南

本專案正持續開發中，程式碼未來可能會有大幅更動。

**目前我們優先審查外部貢獻中的錯誤修正與安全性修補。**

若您想新增功能或變更現有行為，請先開 Issue 提案，並在投入開發前取得 OpenAI 成員的核可。

**未依此流程提出的貢獻可能會被關閉**，若其與目前的路線圖不符，或與其他優先事項/即將推出的功能衝突。

### 開發流程

- 由 `main` 建立「主題分支」——例如 `feat/interactive-prompt`。
- 專注於單一主題。多個不相關的修正應分開提 PR。
- 依照上方［<a href="#development-workflow">開發設定</a>］說明，確保您的變更沒有 lint 警告且測試通過。

### 撰寫高影響力的程式碼變更

1. **從 Issue 開始。** 新開或回覆既有討論，以便在動手寫碼前就解法達成共識。
2. **新增或更新測試。** 每項新功能或錯誤修正都應有對應測試：在變更前失敗、變更後通過。不需 100% 覆蓋率，但需具備有意義的斷言。
3. **撰寫行為文件。** 若變更影響使用者可見的行為，請更新 README、行內說明（`codex --help`）或相關範例專案。
4. **保持提交原子性。** 每個提交都應可編譯且測試通過，方便審查與回滾。

### 建立 Pull Request

- 完整填寫 PR 範本（或提供同等資訊）——**做了什麼？為什麼？怎麼做？**
- 在本機執行「所有」檢查（`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`）。能在本機發現的 CI 失敗若拖到遠端，會延誤時程。
- 確保您的分支已與 `main` 同步，並解決所有衝突。
- 只有當您認為可合併時，才將 PR 標記為 **Ready for review**。

### 審查流程

1. 會有一位維護者擔任主要審查者。
2. 若 PR 新增了未事先討論並核可的新功能，我們可能會關閉該 PR（見［<a href="#contributing">貢獻指南</a>］）。
3. 我們可能會要求修改——請別介意。我們重視您的貢獻，同時也重視一致性與長期可維護性。
5. 當多方同意 PR 達到標準後，維護者會以 squash-and-merge 合併。

### 社群價值

- **友善與包容。** 尊重他人；我們遵循 [Contributor Covenant](https://www.contributor-covenant.org/)。
- **善意假設。** 書面溝通不易——請傾向寬厚以對。
- **教學相長。** 若發現令人困惑的部分，歡迎開 Issue 或 PR 改善。

### 取得協助

若您在設定專案時遇到問題、想為點子徵求回饋，或只是打聲招呼，請開啟 Discussion 或參與相關 Issue。我們樂於協助。

讓我們一起打造更出色的 Codex CLI。**玩得開心！** :rocket:

### 貢獻者授權協議（CLA）

所有貢獻者**必須**接受 CLA。流程相當輕量：

1. 開啟您的 PR。
2. 貼上以下留言（若之前簽署過，回覆 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 機器人會在儲存庫記錄您的簽署，並標記狀態檢查為通過。

無需特別的 Git 指令、電子郵件附件或提交頁尾。

#### 快速修正

| 情境               | 指令                                             |
| ----------------- | ------------------------------------------------ |
| 修正上一筆提交     | `git commit --amend -s --no-edit && git push -f` |

在每筆提交都帶有頁尾之前（squash 時只需最後一筆），**DCO 檢查**會阻擋合併。

### 發佈 `codex`

（僅限管理者）

請確認目前位於 `main` 且沒有本機變更，然後執行：

```shell
VERSION=0.2.0  # 也可以是 0.2.0-alpha.1 或任何有效的 Rust 版本字串。
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這會在 `main` 之上建立一個本機提交，將 `codex-rs/Cargo.toml` 裡的 `version` 設為 `$VERSION`（注意在 `main` 分支上，我們維持 `version = "0.0.0"`）。

接著會以 `rust-v${VERSION}` 標籤推送該提交，並觸發［<a href="../.github/workflows/rust-release.yml">發佈工作流程</a>］，建立名為 `$VERSION` 的 GitHub Release。

若自動產生的 GitHub Release 一切正常，請取消勾選 **pre-release**，讓它成為最新版本。

建立 PR 以更新 Homebrew 上的 [`Formula/c/codex.rb`](https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb)。

### 安全與負責任的 AI

若您發現任何漏洞或對模型輸出有所疑慮，請寄信至 **security@openai.com**，我們會儘速回覆。
