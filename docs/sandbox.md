## 沙盒與核准

### Approval modes

We've chosen a powerful default for how Codex works on your computer: `Auto`. In this approval mode, Codex can read files, make edits, and run commands in the working directory automatically. However, Codex will need your approval to work outside the working directory or access network.

When you just want to chat, or if you want to plan before diving in, you can switch to `Read Only` mode with the `/approvals` command.

If you need Codex to read files, make edits, and run commands with network access, without approval, you can use `Full Access`. Exercise caution before doing so.

#### Defaults 和 recommendations

- codex runs 在 a sandbox 透過 default 使用 strong guardrails: 它 prevents editing 檔案為 codex 提供額外的指示和指導 outside the workspace 和 blocks network access unless enabled.
- 在 launch, codex detects whether the folder is version-controlled 和 recommends:
  - Version-controlled folders: `Auto` (workspace write + on-request approvals)
  - Non-version-controlled folders: `Read Only`
- The workspace includes the current directory and temporary directories like `/tmp`. Use the `/status` command to see which directories are in the workspace.
- 您 can set 這些 explicitly:
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### Can I 執行 without ANY approvals?

Yes, you can disable all approval prompts with `--ask-for-approval never`. This option works with all `--sandbox` modes, so you still have full control over Codex's level of autonomy. It will make its best attempt with whatever contrainsts you provide.

### Common sandbox + approvals combinations

| Intent                                  | 標誌                                                                                  | Effect                                                                                  |
| --------------------------------------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Safe read-only browsing                 | `--sandbox read-only --ask-for-approval on-request`                                            | Codex can read files and answer questions. Codex requires approval to make edits, run commands, or access network. |
| Read-only non-interactive (CI)          | `--sandbox read-only --ask-for-approval never`                                                 | Reads only; never escalates                                                                     |
| Let it edit the repo, ask if risky      | `--sandbox workspace-write --ask-for-approval on-request`                                      | Codex can read files, make edits, and run commands in the workspace. Codex requires approval for actions outside the workspace or for network access. |
| Auto (preset)                           | `--full-auto` (equivalent to `--sandbox workspace-write` + `--ask-for-approval on-failure`)     | Codex can read files, make edits, and run commands in the workspace. Codex requires approval when a sandboxed command fails or needs escalation. |
| YOLO (not recommended)                  | `--dangerously-bypass-approvals-and-sandbox` (alias: `--yolo`)                                 | No sandbox; no prompts                                                                          |

> Note: In `workspace-write`, network is disabled by default unless enabled in config (`[sandbox_workspace_write].network_access = true`).

#### Fine-tuning in `config.toml`

```toml
# approval mode
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# full-auto mode
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# Optional: allow network in workspace-write mode
[sandbox_workspace_write]
network_access = true
```

您 can also save presets 作為 **profiles**:

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### Experimenting 使用 the codex Sandbox

到 測試 到 see 會發生什麼 當 a 指令 is 執行 under the sandbox provided 透過 codex, we provide the following subcommands 在 codex CLI:

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### Platform sandboxing details

The mechanism codex uses 到 implement the sandbox policy depends 在 您的 OS:

- **macOS 12+** uses **Apple Seatbelt** and runs commands using `sandbox-exec` with a profile (`-p`) that corresponds to the `--sandbox` that was specified.
- **Linux** uses a combination of Landlock/seccomp APIs to enforce the `sandbox` configuration.

Note that when running Linux in a containerized environment such as Docker, sandboxing may not work if the host/container configuration does not support the necessary Landlock/seccomp APIs. In such cases, we recommend configuring your Docker container so that it provides the sandbox guarantees you are looking for and then running `codex` with `--sandbox danger-full-access` (or, more simply, the `--dangerously-bypass-approvals-and-sandbox` flag) within your container. 