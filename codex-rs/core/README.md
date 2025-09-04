# codex-core

這個 crate implements the business logic 為 codex. 它 is designed 到 be used 透過 the various codex UIs written 在 Rust.

## Dependencies

Note that `codex-core` makes some assumptions about certain helper utilities being available in the environment. Currently, this

### macOS

Expects `/usr/bin/sandbox-exec` to be present.

### Linux

Expects the binary containing `codex-core` to run the equivalent of `codex debug landlock` when `arg0` is `codex-linux-sandbox`. See the `codex-arg0` crate for details.

### All Platforms

Expects the binary containing `codex-core` to simulate the virtual `apply_patch` CLI when `arg1` is `--codex-run-as-apply-patch`. See the `codex-arg0` crate for details.
