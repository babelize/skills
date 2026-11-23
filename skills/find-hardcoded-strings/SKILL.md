---
name: find-hardcoded-strings
description: Audit a codebase for hardcoded user-facing text that isn't translatable yet, without changing anything. Lists every string per file and line for React, Next.js, Vue, React Native, Django templates and HTML, and estimates the work to internationalize. Use when the user asks how much work i18n would be, wants an i18n readiness audit, asks where hardcoded text is, or wants to check that new code didn't add untranslated strings.
---

# Find hardcoded strings

Read-only: this skill changes no files. It uses the scanner of the Babelize CLI (`npx babelize`, open source). Needs Node 18+.

## Run it

```bash
npx babelize scan .
npx babelize scan . --json            # machine-readable, for counting and grouping
npx babelize scan . --include 'src/**' --exclude 'src/legacy/**'
npx babelize scan . -e .tsx,.vue       # only some extensions
```

It prints each file with the line and text of every hardcoded string it would wrap: JSX text, attributes like `placeholder`, `title`, `alt` and `aria-label`, and template text.

## Report

Summarize for the user:

- Total strings and files, and the top files by count.
- Which areas they're in (pages, components, emails, server code).
- Strings that probably should **not** be translated: brand names, values the code compares (`'active'`, `'off'`), API parameters, LLM prompts. A scanner is pattern-based and lists those too.
- Whether i18n already exists (locale files, an i18n library in `package.json`). If it does, the remaining strings are the ones new code added.

If the user wants to go ahead, the `localize-app` skill does the wrapping, wiring and translation. To keep new hardcoded text out, the `i18n-ci-check` skill adds a check to CI.
