# TestRooster Actions 🐓

GitHub Actions for [TestRooster](https://github.com/TestRooster) — the
orchestration control plane for test execution. These are thin shims that
run inside **your** CI: they authenticate to TestRooster via GitHub Actions
OIDC (no API keys to manage), fetch the **Blast Radius** selection manifest,
run the selected tests, and report results back. All product logic lives
server-side.

Supported runners: **pytest** (Python) and **gotest** (Go). More coming —
.NET is next.

## `select` — run only the tests your change can break

Add to your existing `pull_request` workflow:

```yaml
permissions:
  contents: read
  id-token: write          # OIDC exchange with TestRooster

steps:
  - uses: actions/checkout@v4
    with: { fetch-depth: 0 }    # the diff needs the PR base
  - uses: TestRooster/actions/select@v1
    with:
      api-url: https://your-testrooster-host
      src-dir: src/mypkg        # pytest only
      # runner: gotest          # for Go projects
```

In **shadow mode** (the default for new installs) the full suite still runs
— TestRooster reports what it *would* have selected and its measured
would-be miss-rate before anything is ever skipped.

Inputs: `api-url` (required) · `runner` (`pytest`|`gotest`) · `src-dir` ·
`test-dir` · `trigger` (`pr`|`nightly`) · `budget-ms` · `core-tests` ·
`pytest-args` · `junit-path`.

## `coverage` — the nightly snapshot that powers selection

```yaml
on:
  schedule: [{ cron: '0 7 * * *' }]
  workflow_dispatch:

steps:
  - uses: actions/checkout@v4
  - uses: TestRooster/actions/coverage@v1
    with:
      api-url: https://your-testrooster-host
      # runner: gotest
```

Runs the full suite with per-test coverage and uploads the snapshot the
reverse index is built from. This is also the full-suite safety net that
makes selection safe: a selection miss costs hours of latency, never a
shipped bug.

---

Source of truth for these actions lives in the main TestRooster repository;
this repo is a published mirror. Issues → the TestRooster org.
