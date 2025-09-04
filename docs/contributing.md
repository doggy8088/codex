## 貢獻

此專案正在積極開發中，程式碼可能會發生重大變化。

**目前，我們只計劃使用 Codex 來優先審查針對錯誤或安全修復的外部貢獻。**

如果您想新增新功能或變更現有功能的行為，請在花時間建構之前，先開啟一個議題提議該功能並獲得 OpenAI 團隊成員的批准。

**沒有經過此流程的新貢獻可能會被關閉**，如果它們與我們目前的路線圖不符或與其他優先事項/即將推出的功能衝突。

### 開發工作流程

- 從 `main` 建立 _主題分支_ - 例如 `feat/interactive-prompt`。
- 保持您的變更專注。多個不相關的修復應該作為單獨的 PR 開啟。
- 按照上述 [開發設置](#development-workflow) 說明，確保您的變更沒有 lint 警告和測試失敗。

### 撰寫高影響力的程式碼變更

1. **從議題開始。** 開啟新的議題或在現有討論中留言，以便我們在撰寫程式碼之前就解決方案達成一致。
2. **新增或更新測試。** 每個新功能或錯誤修復都應該包含測試覆蓋，在您的變更之前失敗，之後通過。不需要 100% 覆蓋率，但要有意義的斷言。
3. **記錄行為。** 如果您的變更影響面向使用者的行為，請更新 README、內聯說明（`codex --help`）或相關的範例專案。
4. **保持原子性提交。** 每次提交都應該能夠編譯且測試應該通過。這使審查和潛在的回滾更容易。

### 開啟拉取請求

- 填寫 PR 模板（或包含類似資訊）- **什麼？為什麼？如何？**
- 在本地運行**所有**檢查（`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`）。可以在本地發現的 CI 失敗會拖慢流程。
- 確保您的分支與 `main` 保持最新，並且您已解決合併衝突。
- 只有當您認為它處於可合併狀態時，才將 PR 標記為**準備審查**。

### 審查流程

1. One maintainer will be assigned 作為 a primary reviewer.
2. If your PR adds a new feature that was not previously discussed and approved, we may choose to close your PR (see [Contributing](#contributing)).
3. We may ask 為 changes - please do not take 這個 personally. We value the work, but we also value consistency 和 long-term maintainability.
5. 當 there is consensus 那個 the PR meets the bar, a maintainer will squash-和-merge.

### Community values

- **Be kind and inclusive.** Treat others with respect; we follow the [Contributor Covenant](https://www.contributor-covenant.org/).
- **Assume good intent.** Written communication is hard - err 在 the side 的 generosity.
- **Teach & learn.** 如果 您 spot something confusing, open an issue 或 PR 使用 improvements.

### Getting help

如果 您 執行 into problems setting up the project, would like feedback 在 an idea, 或 just want 到 say _hi_ - please open a Discussion 或 jump into the relevant issue. We are happy 到 help.

Together we can make codex CLI an incredible tool. **Happy hacking!** :rocket:

### Contributor 授權 agreement (CLA)

All contributors **must** accept the CLA. The process is lightweight:

1. Open 您的 pull request.
2. Paste the following comment (or reply `recheck` if you've signed before):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. The CLA-Assistant bot records 您的 signature 在 the repo 和 marks the status check 作為 passed.

No special git commands, email attachments, 或 commit footers required.

#### Quick fixes

| Scenario          | 指令                                          |
| ----------------- | ------------------------------------------------ |
| Amend last commit | `git commit --amend -s --no-edit && git push -f` |

The **DCO check** blocks merges until every commit 在 the PR carries the footer (使用 squash 這個 is just the one).

### Releasing `codex`

_For admins only._

Make sure you are on `main` and have no local changes. Then run:

```shell
VERSION=0.2.0  # Can also be 0.2.0-alpha.1 or any valid Rust version.
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

This will make a local commit on top of `main` with `version` set to `$VERSION` in `codex-rs/Cargo.toml` (note that on `main`, we leave the version as `version = "0.0.0"`).

This will push the commit using the tag `rust-v${VERSION}`, which in turn kicks off [the release workflow](../.github/workflows/rust-release.yml). This will create a new GitHub Release named `$VERSION`.

如果 everything looks good 在 the generated GitHub Release, uncheck the **pre-release** box so 它 is the latest release.

Create a PR to update [`Formula/c/codex.rb`](https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb) on Homebrew.

### Security & responsible AI

Have you discovered a vulnerability or have concerns about model output? Please e-mail **security@openai.com** and we will respond promptly. 