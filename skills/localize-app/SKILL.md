---
name: localize-app
description: Make a React, Next.js, Vue, React Native/Expo, Django or static HTML app multilingual, or bring its out-of-date translation files back in sync. Finds hardcoded UI text, wires up the right i18n library (next-intl, react-i18next, vue-i18n…), translates into the requested languages with the Babelize CLI, then reviews the diff so prompts, data values and logic strings stay untouched, and runs the project's own typecheck, lint and tests. Use when the user asks to translate, localize or internationalize their app, add a language, set up i18n, fix missing or untranslated keys in locale files, or wants to see what it would look like (offline demo, no key).
---

# Localize an app

Translating an app is two jobs: wrapping every user-facing string in the code, and keeping the locale files in sync as the app changes. This skill does both with the Babelize CLI (`npx babelize`, open source), then does the part a tool can't: deciding which wrapped strings were never UI to begin with.

The CLI needs Node 18+ and a git repository. It refuses to run on uncommitted changes (`--force` overrides), so the user's work is never mixed into the result.

## Pick the case first

- **No i18n yet** (hardcoded text, no locale files): extract, wire and translate. Go to the workflow.
- **i18n already set up, translations behind** (`en.json` has keys that `de.json` lacks): run `npx babelize check` first. It lists missing keys, broken placeholders and ICU errors per locale, and exits 1 on errors. Then `translate` with the existing locales fills only the gaps: a lockfile (`.babelize/local/babelize-lock.json`) makes re-runs translate only new or changed strings.

## Who translates

Pick one; ask only if the user's request doesn't settle it.

- **You, the agent (default when there's no key).** Babelize does the extraction and wiring with `--provider pseudo`, `babelize check --json` lists every string still to translate, and you write the translations into the locale files yourself (see "Translating it yourself" below). No key, no account, nothing to pay for, and the translations follow the conversation's context.
- **The user's own LLM key**, for big apps where the CLI's batched translation is faster: `--provider anthropic` (reads `ANTHROPIC_API_KEY`), `openai`, `gemini`, `groq` or `openrouter` (`<NAME>_API_KEY`). No Babelize account needed.
- **Babelize-hosted (free tier):** `npx babelize auth login` (opens the browser) or `BABELIZE_API_KEY` in the environment, then `--provider babelize`.
- **Just a preview:** `--provider pseudo` and stop after the review; the fake text (`⟦Sávé⟧`) must never be committed as translations.

**Never ask the user to paste an API key into chat.** Have them export it in their shell.

## Workflow

1. **Scan.** `npx babelize scan .` (add `--include 'src/**'` or `--exclude` for monorepos and component libraries) lists the hardcoded strings per file. Give the user the count. If they only asked what it would take, stop here.
2. **Keep non-UI code out.** If the project has folders that are clearly not UI (server, API clients, search or query builders, parsers, prompt templates), put them in `.babelize/config.json` before translating, so no run ever touches them:

   ```json
   { "exclude": ["server/**", "utils/web/**", "lib/prompts/**"] }
   ```

   The CLI refuses to run on uncommitted changes, so after creating or editing this file pass `--force` to the next run (or let the user commit it first).

3. **Preview.** `npx babelize translate --to fr,de --provider pseudo --dry-run` prints the diff and writes nothing. Preview with `pseudo` even when the real run will use a key: a dry run with a real provider still calls the translator and spends tokens. Use real locale codes (`pt-BR`, `zh-Hans`). For formal/informal "you" add `--formality de=formal,it=informal`; for tone, `--brand-voice "Friendly, concise"`.
4. **Run.** `npx babelize translate --to fr,de --provider <name> --yes` (`pseudo` when you'll translate it yourself). It wraps the strings, and when the i18n setup is missing it also edits the app's entry files (root layout, framework config, `package.json`) and adds config files and locale files, so expect more changed files than the scan listed. `--allow-partial` keeps going when a few keys fail; those are retried on the next run.
5. **Review the diff** with the checklist below. This is the step that matters most. Revert what doesn't belong by hand, and remove the matching keys from every locale file, the source one included.
6. **Install and lock.** The CLI edits `package.json`, not the lockfile. Run the project's own package manager (`npm install`, `pnpm install`, `yarn install`) so the lockfile matches, or CI with a frozen lockfile fails.
7. **Run the project's checks:** its typecheck, linter (and formatter, so quotes and spacing match the codebase) and tests. Note failures that already happened before the change rather than fixing them. Two common snags:
   - Next.js: `tsc` fails on types like `LayoutProps` until Next has generated them. Run `next build` (or `next typegen`) first.
   - Component tests that render translated text need the i18n provider in their setup: for react-i18next, import the i18n config in the test setup file; for next-intl, wrap in `<NextIntlClientProvider locale="en" messages={en}>`.
8. **Translate, if it's on you:** follow "Translating it yourself" below.
9. **`npx babelize check`** must pass with 0 errors. The only exception is a pure preview, where `pseudo_translation` errors are expected; any other error type is a real problem.
10. **Report:** strings wrapped, languages, files changed, what you reverted and why, check results, and anything left for the user (below). Commit only if they ask, and include `.babelize/local/babelize-lock.json` with real translations so re-runs stay incremental.

## Translating it yourself

Order matters, because each `translate` run records keys in the lockfile and a key the lockfile doesn't know yet is treated as new:

1. **Finish every code edit first** (steps 4–7): reverts, plural rewrites, anything that adds or removes keys.
2. **Run once more:** `npx babelize translate --to fr,de --provider pseudo --yes --force`. Every key now exists and has pseudo text. Look at the diff again: if it re-wrapped something you reverted, exclude that file (see the checklist).
3. **List what's left:** `npx babelize check --json`. Issues are under `sets[].locales[].issues[]`; each one with `"code": "pseudo_translation"` or `"missing_key"` has the `locale` and `key` to fill. The English text is the same key in the source locale file.
4. **Translate** by editing the target locale files directly (`public/locales/fr.json`…), one locale at a time, keeping the file's structure, key order and formatting. Translate from the source text, never from the pseudo text. Keep exactly:
   - `{placeholders}` and their names, rich-text tags like `<b>` or `<link>`, and leading or trailing spaces and punctuation the UI relies on.
   - ICU syntax, with **every plural category the target language uses**, not just the source's: `ja`/`zh`/`ko`: `other`; `de`/`en`/`nl`/`sv`/`it`/`es`/`pt`: `one`, `other` (`es`, `it` and `pt` also use `many` for large round numbers); `fr`: `one`, `many`, `other`; `ru`/`uk`/`pl`: `one`, `few`, `many`, `other`; `ar`: `zero`, `one`, `two`, `few`, `many`, `other`. The pseudo text already lists the categories Babelize expects; keep them all.
5. **Follow the project's settings** in `.babelize/config.json`: `glossary` (`neverTranslate` terms stay as they are, including a brand name that is a whole string; `alwaysTranslateAs` gives the exact translation), `formality` and `brandVoice`. If a language has formal and informal "you" (de, fr, es, it, nl, pt…) and there's no setting and no existing translation to follow, ask the user once; if they don't mind, use formal.
6. **Record them:** run `npx babelize translate --to fr,de --provider pseudo --yes --force` one last time. It writes nothing new; it records your translations in the lockfile as external, so later runs keep them. (One exception: a plural message you created by hand during review may be re-translated by a later run with a real provider.) Then `npx babelize check` must report 0 errors.

On later runs repeat 2–6: only new or changed strings get pseudo text.

## Review checklist

Extraction is pattern-based, so it also wraps strings that look like text but are data or logic. Translating those changes behaviour. **Revert any wrap of:**

- **Prompts and model input:** system prompts, `{ role: 'system', content: … }`, instructions sent to an LLM, language names or word lists used to build prompts or detect a language.
- **Values the code compares or passes on:** state and enum values (`'front'`, `'off'`), API parameters, query strings, search terms, event names, storage keys, `testID`s, CSS values and font names (`'Menlo'`), URLs, MIME types, regexes.
- **Language-keyed data:** maps like `{ pl: '…', en: '…' }` that already hold one string per language.
- **Non-UI modules** that slipped past the exclude list: log and analytics messages, developer-only validation errors. Add their folders to `exclude`.
- **Brand and product names** are not reverted: a run would wrap them again. Leave the wrap, keep the value identical to the source in every locale, and add the name to the glossary in `.babelize/config.json` so sentences that contain it keep it too:

  ```json
  { "glossary": { "neverTranslate": ["Tasklane"], "alwaysTranslateAs": [{ "term": "Workspace", "translations": { "de": "Arbeitsbereich" } }] } }
  ```

  With a real provider, run `translate` again afterwards: keys whose glossary rules changed are re-translated.

**A reverted string inside a UI file can be wrapped again on the next run**, because there is no per-string opt-out. Exclude whole files where you can, and repeat this review after every run.

**Check the code changes too:**

- A hook (`useTranslation`, `useTranslations`) must sit at the top of a component, never inside a callback or render prop; reuse the component's `t`.
- Strings at module scope are evaluated once at import time and won't follow a language switch. Keep the key there and call `t` where the value is used.
- Counts written as two strings (``n === 1 ? '1 task' : `${n} tasks` ``) become two keys, which is wrong for languages with other plural rules. Rewrite as one ICU message: `{count, plural, one {# task} other {# tasks}}`.
- Keys reported as "not wired into code" are translated but unused. Drop them unless the user wants to wire them by hand.
- Follow the project's own rules over the generated code (quote style, no explanatory comments, import order).

**Check what extraction doesn't cover**, and tell the user if you didn't fix it:

- Page metadata: `<title>`, `metadata` / `generateMetadata` in Next.js, meta descriptions, Open Graph text.
- `<html lang>` (and `dir` for right-to-left languages) should change with the locale. Open a translated page or route to check.
- **Right-to-left layout** (`ar`, `he`, `fa`, `ur`): translated strings aren't enough if the CSS still says `margin-left`, `padding-right`, `left: 0` or `text-align: left`; the RTL version comes out mirrored wrong. Point out physical properties and suggest logical ones (`margin-inline-start`, `padding-inline-end`, `inset-inline-start`, `text-align: start`; Tailwind `ms-*`, `pe-*`, `start-*`). Directional icons like arrows may need flipping.
- Text in images, emails, server-rendered error pages and anything built from data.

## Framework notes

What Babelize sets up, and what to check, per framework:

- **Next.js (App Router):** next-intl, with `i18n/request.ts`, `i18n/locales.ts` and a `proxy.ts` (middleware before Next 16) that serves `/fr/...` from the existing routes; the source language stays unprefixed. `"routing": "cookie"` in `.babelize/config.json` keeps one URL per page instead. Components get `useTranslations` (also fine in Server Components). `<html lang>` and page `metadata` stay in the source language: tell the user. Setting `lang` from `getLocale()` in the root layout can break `next build` in projects with `cacheComponents` on, so don't change it without building.
- **React (Vite, CRA):** react-i18next. The i18n config must be imported once at the entry point, before the app renders.
- **React Native / Expo:** locales are bundled in `src/locales` (never under Expo Router's `app/`, which would turn them into routes) and the device language comes from `expo-localization` or `react-native-localize`, imported first. Tests need the i18n config loaded in the Jest setup file.
- **Vue:** vue-i18n with `legacy: false` (Composition API, `useI18n`), wired in `main.*`.
- **Django:** gettext. Templates and Python strings are marked in place, translations go to `locale/<lang>/LC_MESSAGES/django.po` with the compiled `.mo` next to it, and `settings.py` gets the languages. If the project's deploy compiles messages itself and ignores `.mo` files in git, follow that.
- **Static HTML:** a small runtime script swaps text by key and adds a language switcher; locale JSON files sit next to the pages.

If the project already has its own i18n setup, Babelize uses it as is and changes no code; that case is the `sync-translations` skill.

## Don't

- Don't push, open a pull request or publish anything without the user asking.
- Don't commit while any `⟦…⟧` pseudo text is left in the locale files, and don't fill a key with the English text to make `check` pass; translate it or leave it failing.
- Don't change the i18n library a project already uses.
