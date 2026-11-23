---
name: auto-translate-on-push
description: Keep an app's translations up to date automatically - after every push to a branch, the new and changed strings are translated and delivered as one pull request that stays updated. Sets this up with the Babelize GitHub App (free plan included) and explains what the PR contains and how to review it. Use when the user wants translations to stay in sync without manual work, continuous localization, or a translation PR on every push.
---

# Translate on every push

Babelize (app.babelize.co, open-source CLI) watches a GitHub branch. A few minutes after each push it translates what's new or changed and opens or updates a single pull request with the locale files (and, on the first run, the code changes). The team reviews and merges it like any other PR. This works on the free plan.

The setup happens in the browser and on GitHub, so walk the user through it; you can't click for them.

## Set up

1. Sign up at https://app.babelize.co.
2. **New project → GitHub.** When asked, install the Babelize GitHub App on the repository; it needs push access to open the pull request.
3. Pick the branch to watch (usually `main`) and the target languages.
4. Keep **"Keep translating when I push to `<branch>`"** checked. Start the first run.

After that, every push to the branch is picked up automatically. Pushes within a few minutes of each other are batched into one run.

## The first pull request

- **App without i18n yet:** the first PR wraps hardcoded strings, adds the i18n setup and the locale files. Review it like code: the `localize-app` skill's review checklist applies (strings that aren't UI, such as prompts, state values and font names, must not be wrapped). The Babelize dashboard shows every translation and lets the user fix wording before the PR is updated.
- **App with i18n already:** the PR only adds missing keys and new languages to the existing locale files; existing translations are kept.

## Day to day

- New PRs aren't opened for every push: the same PR is updated until it's merged.
- A language added in the project settings joins the open PR on the next run.
- Glossary (terms never translated, or always translated a set way), brand voice and formal/informal address are set per project in the dashboard and apply to every run.

To also stop broken translations from being merged by hand, add the `i18n-ci-check` skill's CI step. To run translations locally instead of on push, see `sync-translations`.
