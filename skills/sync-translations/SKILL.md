---
name: sync-translations
description: Fill missing translations and add new languages in an app that already has i18n (react-i18next, next-intl, vue-i18n, Django gettext, Expo/React Native). Finds keys missing per language, broken {placeholders} and invalid ICU plurals, translates only the gaps, keeps every existing translation as it is, and changes no code. Use when translations are out of date, some text still shows in English, a locale file is missing keys, or the user wants to add a language such as Spanish, Japanese or Arabic.
---

# Sync translations and add languages

Apps change faster than their translations: every feature adds keys to the source locale file and the other languages fall behind. This skill finds exactly what's missing or broken and fills only that, with the Babelize CLI (`npx babelize`, open source). Needs Node 18+ and a git repo with a clean working tree.

If the app has no i18n yet (text hardcoded in components), use the `localize-app` skill instead.

## 1. Find the gaps

```bash
npx babelize check
```

Per language it reports missing keys, failed strings, placeholders that don't match the source (`{name}` missing or renamed), invalid ICU plurals and glossary violations. It exits 1 on errors; `--json` for a full report, `--locales de,fr` to limit it. The source locale is `en` unless `.babelize/config.json` sets `"sourceLocale"` (or pass `--from`).

Tell the user the counts per language.

## 2. Fill them, or add a language

With a key, the CLI does it in one command:

```bash
npx babelize translate --to de,fr --provider <name> --yes        # the project's languages
npx babelize translate --to de,fr,es --provider <name> --yes     # plus a new one
```

List every language the project has, plus any new one. Don't pass `--replace`: it deletes locales that aren't listed.

**What it does with an existing setup:** it reads the project's own i18n config and locale files, keeps every translation already there (including ones edited by hand), translates only missing keys, and writes the new language's file in the same layout as the others. It changes no code and no config, and says so ("Used the project's existing … setup"). A lockfile (`.babelize/local/babelize-lock.json`) then keeps later runs incremental: commit it.

One exception: in a setup that Babelize itself generated earlier, don't delete that lockfile. Without it, the next run can't tell hand edits apart and re-translates everything.

Who translates (ask only if the request doesn't settle it):

- **You, the agent (default when there's no key):** skip `translate` for the missing keys. `npx babelize check --json` lists each one under `sets[].locales[].issues[]` (`"code": "missing_key"`, with `locale` and `key`); add the translation to that locale file yourself, from the source locale's text, keeping the file's structure and key order, every `{placeholder}`, ICU plural syntax with every plural category the target language uses (fr: one/many/other; de, es: one/other plus `many` for es; ru, pl: one/few/many/other; ar: all six; ja, zh: other) and rich-text tags. For a new language, copy the source file's structure into the new file and translate every value. Then `check` again.
- **The user's own LLM key**, faster for many keys: `--provider anthropic` (reads `ANTHROPIC_API_KEY`), `openai`, `gemini`, `groq` or `openrouter` (`<NAME>_API_KEY`).
- **Babelize's free hosted tier:** `npx babelize auth login` or `BABELIZE_API_KEY`, then `--provider babelize`.

Never ask the user to paste a key into chat.

Match the existing translations' style:
- Formal or informal "you": `--formality de=formal,es=informal`, or `"formality"` in `.babelize/config.json`.
- Terms: `"glossary": { "neverTranslate": ["Acme"], "alwaysTranslateAs": [{ "term": "Workspace", "translations": { "de": "Arbeitsbereich" } }] }` in `.babelize/config.json`.

## 3. A new language: register it

Babelize writes the locale file. If the project's i18n was set up by hand, the code may also list its languages: an `i18next` `resources` or `supportedLngs`, next-intl `locales`, vue-i18n `messages`, Django `LANGUAGES` in `settings.py`, a language switcher. Add the new one there and show the user the change.

Right-to-left languages (`ar`, `he`, `fa`, `ur`): check that `<html dir="rtl">` (or `I18nManager` on React Native) follows the locale and that styles use logical properties (`margin-inline-start`, not `margin-left`). Tell the user what isn't RTL-ready.

## 4. Verify

`npx babelize check` should pass. The diff should only touch locale files (and the lockfile), plus any registration you added. Run the project's tests if they render translated text. Don't fill failed keys with the source text, and don't commit unless the user asks.
