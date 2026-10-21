# Local CI signoff preparation

Hosted CI stays enabled. This is preparation for card 248, not approval to
remove workflows or merge. Fizzy converts last, after the simpler pilots.

`bin/ci` runs setup and checks. Setup can change local state. It no longer
posts a GitHub status. Do not use it as an isolated test runner. A passing
run does not prove the full database matrix. Run each gate in an isolated
test environment and retain logs outside this checkout.

## Gate map

The proposed names below are explicit gh-signoff context arguments. GitHub
adds `signoff/`. They are not installed or attested by this change.

| Hosted gate or extra local gate | Proposed context | Local command or requirement |
| --- | --- | --- |
| Checks / Gemfile drift | fizzy-drift | `bin/bundle-drift check` |
| Checks / Security | fizzy-security | `bin/bundler-audit check --update`, `bin/importmap audit`, `bin/brakeman --quiet --no-pager --exit-on-warn --exit-on-error` |
| Local secret scan | fizzy-gitleaks | `bin/gitleaks-audit` |
| Checks / Lint | fizzy-lint | `bin/rubocop` |
| Checks / Lint, agent docs | fizzy-agent-docs | `script/check_agents_docs` |
| Checks / GitHub Actions audit | fizzy-actions | `zizmor --no-progress .github/workflows` |
| Setup contract | fizzy-setup | `test/setup-phases-test` (synthetic command stubs) |
| OSS / SQLite | fizzy-oss-sqlite | `bin/rails db:setup test` |
| OSS / SQLite system | fizzy-oss-sqlite-system | `PARALLEL_WORKERS=1 bin/rails test:system` |
| OSS / MySQL 8.4 | fizzy-oss-mysql84 | `bin/rails db:setup test` |
| OSS / MySQL 8.4 system | fizzy-oss-mysql84-system | `PARALLEL_WORKERS=1 bin/rails test:system` |
| SaaS / SQLite | fizzy-saas-sqlite | `bin/rails db:setup test` |
| SaaS / SQLite system | fizzy-saas-sqlite-system | `PARALLEL_WORKERS=1 bin/rails test:system` |
| SaaS / MySQL 8.4 | fizzy-saas-mysql84 | `bin/rails db:setup test` |
| SaaS / MySQL 8.4 system | fizzy-saas-mysql84-system | `PARALLEL_WORKERS=1 bin/rails test:system` |

Use `RAILS_ENV=test` and `CI=1` for Rails. Set `SAAS=false
BUNDLE_GEMFILE=Gemfile` for OSS, or `SAAS=true BUNDLE_GEMFILE=Gemfile.saas`
for SaaS. Select `DATABASE_ADAPTER=sqlite` or `mysql` explicitly. SaaS needs
its own instructions and private dependencies before execution.

The reusable `test.yml` defines both database adapters for both modes.
`ci-saas.yml` runs OSS on push, but its SaaS job requires repository owner
`basecamp`. It skips SaaS on this personal fork. `ci-oss.yml` runs only for
PRs whose head repository differs from the target. A same-repository draft
therefore uses the push workflow for OSS coverage. Workflow definitions do
not establish that the private SaaS mode passed.

## Isolation and dependencies

Use separate disposable databases, directories and browser ports for this
workspace. Existing database defaults are shared names and paths. Do not
run `db:setup` against those defaults without proving they are disposable.
Use an isolated MySQL 8.4 container with a unique name and loopback port;
do not connect to an existing service. Do not install persistent services.
Keep fixture data synthetic. Test mail uses Action Mailer's test transport;
external providers must use simulated transports. Review network mocks
before a full suite run. Never use production credentials or user data.

Required tools include Ruby from `.ruby-version`, Bundler, SQLite, MySQL
8.4, Chrome and its driver, libvips, ffmpeg, mise, gitleaks and zizmor.
Security audits also need current advisory and importmap audit data.
Pin and record the zizmor engine version before claiming parity: hosted
CI pins an action wrapper, not a local engine version.

## Named commit operation

Use the official [gh-signoff v0.4.1 documentation](https://github.com/basecamp/gh-signoff/blob/v0.4.1/README.md).
Verify the installed extension source is v0.4.1 before use. This draft does
not upgrade a shared extension or write GitHub statuses.

For a published full 40-character SHA, after a complete gate passes on
that exact clean tree, the supported command shape is:

```sh
gh signoff --commit "$SHA" fizzy-oss-sqlite
```

On failure, invalidate that gate for the same named SHA:

```sh
gh signoff fail --commit "$SHA" --description "OSS SQLite checks failed" fizzy-oss-sqlite
```

Replace the context only with the matching row above. Do not post success
for missing dependencies, partial runs, changed files, a different commit,
or an unproved mode. Do not use `-f` or a bare `gh signoff`. Keep command,
exit code, environment, tool versions and logs for every gate. Check the
published PR head still equals the tested SHA before posting. A failure
must not leave a previous success as the latest status for that context.

## Final conversion prerequisites

Snapshot workflows, classic protection and rulesets first. Prove all
applicable local gates, then require the explicit contexts on the default
branch with active enforcement, force-push protection and deletion
protection. Read effective branch rules and exact-head statuses to show
that unsigned heads lack required checks for actors without bypass.
Disclose admin bypass and all other bypass actors. Do not test by merging
or bypassing. No rules means no proof of merge blocking.

Wait for pilot success and independent review of the final published SHA.
Only then consider removal of hosted quality gates. Always preserve
`dependabot-sync-saas-lockfile.yml` and `publish-image.yml`. Preserve other
beneficial automation. Keep this PR draft and card open until all required
proof exists. Missing plan access, dependencies or parity is a blocker,
not a reason to weaken the gate.
