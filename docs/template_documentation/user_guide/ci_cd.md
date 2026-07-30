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
- `scheduled_release.yml` checks weekly for unreleased commits, creates a tag,
  and creates a GitHub release. It calls `publish.yml`, which is gated the
  same way by `PUBLISH_TO_CRATES_IO`.

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
