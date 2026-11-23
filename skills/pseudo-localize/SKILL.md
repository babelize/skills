---
name: pseudo-localize
description: Test whether an app's UI is ready for translation with pseudo-localization, offline and with no API key. Replaces every string with accented, bracketed, longer fake text (⟦Sávé⟧) so untranslated hardcoded text, truncation, overflow and broken layouts show up immediately. Use when the user wants to test i18n, check UI with long text, find strings that aren't translatable, or preview localization before paying for translations.
---

# Pseudo-localize

Pseudo-localization swaps every string for a fake "translation" like `⟦Sávé chángés⟧`: readable, recognisable, a bit longer, and full of accents. In the running app, any plain English left over is hardcoded text that was missed, and anything cut off or overflowing will break in German or Finnish too. Nothing leaves the machine and no key is needed.

Uses the Babelize CLI (`npx babelize`, open source). Needs Node 18+ and a git repo with a clean working tree.

## Run it

```bash
npx babelize translate --to fr --provider pseudo --dry-run   # see the diff only
npx babelize translate --to fr --provider pseudo --yes       # write it
```

The target locale is just a slot to fill; pick one the app will support. If the app has no i18n yet, this also wraps the strings and adds the setup, which makes it the cheapest way to see the whole change.

## Look at the result

1. Start the app and switch to that locale: the app's language switcher or setting, or for a Next.js app that Babelize set up, the `/fr/...` URL.
2. Note every string still in plain English: it isn't wrapped. Note clipped buttons, wrapped labels and overflowing cards.
3. `npx babelize check` exits 1 with `pseudo_translation` errors by design; any other error type is real.

## Clean up

Pseudo text must never ship. If the user only wanted a look, discard it: `git checkout -- .` and `git clean -fd` on the files the run created (show them first). If they're going ahead, a real run (`--provider` with a real translator) re-translates every pseudo value automatically. Never commit the pseudo locale files or lockfile.
