## 貢獻

這個 project is under active 開發 和 the code will likely change pretty significantly.

**在 the moment, we only 方案中使用 Codex 到 prioritize reviewing external contributions 為 bugs 或 security fixes.**

如果 您 want 到 add a new feature 或 change the behavior 的 an existing one, please open an issue proposing the feature 和 get approval 來自 an OpenAI Team member before spending time building 它.

**New contributions that don't go through this process may be closed** if they aren't aligned with our current roadmap or conflict with other priorities/upcoming features.

### 開發工作流程

- Create a _topic branch_ from `main` - e.g. `feat/interactive-prompt`.
- Keep 您的 changes focused. Multiple unrelated fixes should be opened 作為 separate PR.
- Following the [development setup](#development-workflow) instructions above, ensure your change is free of lint warnings and test failures.

### 撰寫高影響力的程式碼變更

1. **Start 使用 an issue.** Open a new one 或 comment 在 an existing discussion so we can agree 在 the solution before code is written.
2. **Add 或 更新 tests.** Every new feature 或 bug-fix should come 使用 測試 coverage 那個 fails before 您的 change 和 passes afterwards. 100% coverage is not required, but aim 為 meaningful assertions.
3. **Document behaviour.** If your change affects user-facing behaviour, update the README, inline help (`codex --help`), or relevant example projects.
4. **Keep commits atomic.** Each commit should compile 和 the tests should pass. 這個 makes reviews 和 potential rollbacks easier.

### 開啟拉取請求

- Fill 在 the PR template (或 include similar information) - **什麼? 為什麼? 如何?**
- Run **all** checks locally (`cargo test && cargo clippy --tests && cargo fmt -- --config imports_granularity=Item`). CI failures that could have been caught locally slow down the process.
- Make sure your branch is up-to-date with `main` and that you have resolved merge conflicts.
- Mark the PR 作為 **Ready 為 review** only 當 您 believe 它 is 在 a merge-able state.

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