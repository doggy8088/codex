## 貢獻指南

本專案正在積極開發中，程式碼可能會發生重大變更。

**目前，我們僅優先審核外部針對錯誤或安全性修復的貢獻。**

若您想新增功能或變更現有功能的行為，請先開設一個 issue 來提議該功能，並在花時間建構前獲得 OpenAI 團隊成員的核准。

**未經過此流程的新貢獻，若與我們目前的路線圖不符，或與其他優先事項／即將推出的功能衝突，可能會被關閉。**

### 開發工作流程

- 從 `main` 建立一個 _主題分支_ - 例如 `feat/interactive-prompt`。
- 保持您的變更具備單一焦點。多個不相關的修復應以獨立的 PR 開啟。
- 遵循上述的 [開發設定](#development-workflow) 指示，確保您的變更沒有任何 lint 警告和測試失敗。

### 撰寫高影響力的程式碼變更

1. **從 issue 開始。** 開設一個新的 issue 或在現有討論中留言，以便我們在撰寫程式碼前能對解決方案達成共識。
2. **新增或更新測試。** 每項新功能或錯誤修復都應包含測試覆蓋，該測試在您的變更前會失敗，而在變更後會通過。不要求 100% 的覆蓋率，但目標是進行有意義的斷言。
3. **為行為撰寫文件。** 如果您的變更影響到使用者可見的行為，請更新 README、內嵌說明 (`codex --help`) 或相關的範例專案。
4. **保持 commits 的原子性。** 每個 commit 都應該能夠編譯且測試都能通過。這使得審核和潛在的回滾更加容易。

### 開啟 Pull Request

- 填寫 PR 範本（或包含類似資訊） - **做了什麼？為什麼做？如何做？**
- 在本機執行**所有**檢查 (`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`)。可被本機捕獲的 CI 失敗會拖慢整個流程。
- 確保您的分支與 `main` 保持同步，並且您已解決合併衝突。
- 僅在您認為 PR 處於可合併狀態時，才將其標示為 **Ready for review**。

### 審核流程

1. 將會指派一位維護者作為主要審核人。
2. 如果您的 PR 新增了先前未經討論和核准的功能，我們可能會選擇關閉您的 PR（請參閱 [貢獻指南](#contributing)）。
3. 我們可能會要求您進行修改 - 請不要將此視為針對個人。我們重視您的工作，但我們也同樣重視一致性和長期可維護性。
5. 當達成共識，認為該 PR 已達標準時，維護者將會進行 squash-and-merge。

### 社群價值觀

- **友善與包容。** 尊重他人；我們遵循 [貢獻者契約](https://www.contributor-covenant.org/)。
- **假定善意。** 書面溝通很困難 - 請多一份寬容。
- **教學相長。** 如果您發現任何令人困惑之處，請開設 issue 或 PR 提出改進建議。

### 尋求協助

如果您在設定專案時遇到問題、希望對某個想法獲得回饋，或只是想打聲招呼 _hi_ —— 請開設一個 Discussion 或參與相關的 issue。我們很樂意提供協助。

我們可以一起讓 Codex CLI 成為一個絕佳的工具。 **祝您開發愉快！** :rocket:

### 貢獻者授權協議 (CLA)

所有貢獻者 **必須** 接受 CLA。流程相當輕量：

1. 開啟您的 Pull Request。
2. 貼上以下留言（如果您之前簽署過，請回覆 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 機器人會在 repo 中記錄您的簽署，並將狀態檢查標示為通過。

不需要特殊的 Git 指令、電子郵件附件或 commit 註腳。

#### 快速修復

| 情境              | 指令                                             |
| ----------------- | ------------------------------------------------ |
| 修訂上一個 commit | `git commit --amend -s --no-edit && git push -f` |

**DCO 檢查**會阻擋合併，直到 PR 中的每個 commit 都帶有註腳為止（使用 squash 合併時，就只需要一個）。

### 發佈 `codex`

 _僅供管理員使用。_

請確認您位於 `main` 分支且沒有任何本地變更。然後執行：

```shell
VERSION=0.2.0  # Can also be 0.2.0-alpha.1 or any valid Rust version.
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

這會在 `main` 的基礎上建立一個本地 commit，並將 `codex-rs/Cargo.toml` 中的 `version` 設為 `$VERSION`（請注意，在 `main` 分支上，我們會將版本保留為 `version = "0.0.0"`）。

這會使用標籤 `rust-v${VERSION}` 來推送 commit，進而觸發[發行工作流程](../.github/workflows/rust-release.yml)。這將會建立一個名為 `$VERSION` 的新 GitHub Release。

如果在產生的 GitHub Release 中一切看起來都沒問題，請取消勾選 **pre-release**（預發行）核取方塊，使其成為最新的發行版。

建立一個 PR 來更新 Homebrew 上的 [`Formula/c/codex.rb`](https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb)。

### 安全性與負責任的 AI

您是否發現了漏洞或對模型輸出有疑慮？請寄送電子郵件至 **security@openai.com**，我們將會盡速回覆。 