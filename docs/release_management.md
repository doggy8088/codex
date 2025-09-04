# Release Management

Currently, we made codex binaries available 在 three places:

- GitHub Releases https://github.com/openai/codex/releases/
- `@openai/codex` on npm: https://www.npmjs.com/package/@openai/codex
- `codex` on Homebrew: https://formulae.brew.sh/formula/codex

# Cutting a Release

Currently, choosing the version number for the next release is a manual process. In general, just go to https://github.com/openai/codex/releases/latest and see what the latest release is and increase the minor version by `1`, so if the current release is `0.20.0`, then the next release should be `0.21.0`.

Assuming you are trying to publish `0.21.0`, first you would run:

```shell
VERSION=0.21.0
./codex-rs/scripts/create_github_release.sh "$VERSION"
```

This will kick off a GitHub Action to build the release, so go to https://github.com/openai/codex/actions/workflows/rust-release.yml to find the corresponding workflow. (Note: we should automate finding the workflow URL with `gh`.)

當 the workflow finishes, the GitHub Release is "done," but 您 still have 到 consider npm 和 Homebrew.

## Publishing 到 npm

After the GitHub Release is done, you can publish to npm. Note the GitHub Release includes the appropriate artifact for npm (which is the output of `npm pack`), which should be named `codex-npm-VERSION.tgz`. To publish to npm, run:

```
VERSION=0.21.0
./scripts/publish_to_npm.py "$VERSION"
```

Note that you must have permissions to publish to https://www.npmjs.com/package/@openai/codex for this to succeed.

## Publishing 到 Homebrew

為 Homebrew, we are properly set up 使用 their automation system, so every few hours 或 so 它 will check our GitHub repo 到 see 如果 there is a new release. 當 它 finds one, 它 will put up a PR 到 建立 the equivalent Homebrew release, 哪個 entails building codex CLI 來自 source 在 various versions 的 macOS.

Inevitably, 您 just have 到 refresh 這個 page periodically 到 see 如果 the release has been picked up 透過 their automation system:

https://github.com/Homebrew/homebrew-core/pulls?q=%3Apr+codex

Once everything builds, a Homebrew admin has 到 approve the PR. Again, the whole process takes several hours 和 we don't have total control over 它, but 它 seems 到 work pretty well.

為 reference, our Homebrew formula lives 在:

https://github.com/Homebrew/homebrew-core/blob/main/Formula/c/codex.rb
