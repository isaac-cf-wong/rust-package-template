# CI/CD

GitHub Actions run on pull requests and pushes to `main`.

- `ci.yml` runs formatting, clippy, tests, docs, and audit checks.
- `documentation.yml` publishes the Zensical site to GitHub Pages. The
  `deploy` job only runs when the repository variable `DEPLOY_DOCS` is
  `"true"`.
- `release.yml` creates GitHub releases from tags.
- `publish.yml` always verifies tagged crate releases (version match,
  `cargo package`), but the `publish` job only runs when the repository
  variable `PUBLISH_TO_CRATES_IO` is `"true"`.
- `scheduled_release.yml` checks weekly whether anything that reaches the
  built crate changed since the latest tag (see
  [What counts as a change](#what-counts-as-a-change)); if so, it creates a
  tag and a GitHub release. It calls `publish.yml`, which is gated the same way
  by `PUBLISH_TO_CRATES_IO`.

## What counts as a change

The `check_new_commits` job runs
[detect-git-changes-action](https://github.com/isaac-cf-wong/detect-git-changes-action)
with `preset: rust`. It counts a change only when it can alter the released
crate: the crate's sources and the files they pull in (`include_str!`,
`include_bytes!`, `#[path]` modules), the build script and the inputs it
declares with `cargo:rerun-if-changed`, `Cargo.toml` outside development-only
tables, the readme and license files, and, because the crate ships a binary,
`Cargo.lock` when a dependency the binary is built with changes. Edits to
`[dev-dependencies]`, `[lints]`, tests, benches, examples, docs, CI workflows
and `rust-toolchain.toml` do not, so routine dev-dependency and tooling bumps
don't trigger a release.

To count more files, add them to `paths` on that step; to leave some out, use
`exclude-paths` (for example `exclude-paths: Cargo.lock` to stop lock-file
updates from releasing). If the preset cannot work out a build input, for
example a build script that declares no `rerun-if-changed`, it warns and every
changed file counts.

## Enabling publishing and docs deployment with repository variables

By default the template ships with publishing and documentation deployment
disabled, so a freshly generated repository stays green with no crates.io or
Pages setup. Each gated job checks a repository variable
(**Settings → Secrets and variables → Actions → Variables**) and only runs
when it is set to the string `"true"`:

| Variable               | Gates                                                                         |
| ---------------------- | ----------------------------------------------------------------------------- |
| `PUBLISH_TO_CRATES_IO` | `publish.yml` (`publish`), including when called from `scheduled_release.yml` |
| `DEPLOY_DOCS`          | `documentation.yml` (`deploy`)                                                |

To enable publishing in a real package:

1. Create a `crates-io` GitHub environment.
2. Add a `CARGO_REGISTRY_TOKEN` secret with a crates.io API token.
3. Set the repository variable `PUBLISH_TO_CRATES_IO` to `true`.

To enable documentation deployment:

1. Enable GitHub Pages with source **GitHub Actions**
   (**Settings** → **Pages**).
2. Set the repository variable `DEPLOY_DOCS` to `true`.

Leave a variable unset (or any value other than `"true"`) to keep that job
disabled.
