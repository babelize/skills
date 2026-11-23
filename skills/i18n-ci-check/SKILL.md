---
name: i18n-ci-check
description: Add a CI check that fails a pull request when translations are missing or broken (missing keys, mismatched {placeholders}, invalid ICU plurals) in an app's locale files. Sets up a GitHub Actions workflow (or the equivalent step for GitLab CI and others) running the Babelize CLI's check. Use when the user wants to catch untranslated strings in CI, block merges with broken translations, or add an i18n lint step.
---

# Check translations in CI

`npx babelize check` (Babelize CLI, open source) validates every locale file against the source locale and exits 1 when something is wrong. It needs no account, no key and no network beyond `npx` itself, so it fits in any CI.

## Try it locally first

```bash
npx babelize check
```

Fix or acknowledge what it reports before turning on the gate, or the first CI run fails on old problems. If the project's source locale isn't `en`, set `"sourceLocale"` in `.babelize/config.json`.

## GitHub Actions

Create `.github/workflows/i18n.yml`:

```yaml
name: i18n
on:
  pull_request:
    paths: ['**/locales/**', '**/messages/**', '**/*.po', '.babelize/**']
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npx --yes babelize check
```

Adjust `paths` to where the project keeps its locale files, or drop the filter to run on every PR. In other CI systems, the step is the same: Node 18+, then `npx --yes babelize check`.

## Optional

- `--json` writes a machine-readable report, e.g. to post as a PR comment.
- To also keep new hardcoded strings out, add a step that runs `npx babelize scan . --json` and fails above a count the team agrees on.
- To have missing translations filled automatically instead of only reported, see the `auto-translate-on-push` skill.

Don't push the workflow unless the user asks.
