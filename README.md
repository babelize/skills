<div align="center">

# Babelize skills

**Agent skills that make your app multilingual, and keep it that way.**

Works with Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and [40+ other agents](https://github.com/vercel-labs/skills#supported-agents) that read `SKILL.md`.

```bash
npx skills add babelize/skills
```

or, in Claude Code: `/plugin marketplace add babelize/skills` then `/plugin install babelize@babelize`

</div>

---

## Skills

| Skill | Ask your agent | What it does |
|---|---|---|
| [**localize-app**](skills/localize-app/SKILL.md) | *"Translate my app into German and French"* | Finds hardcoded UI text, wraps it, sets up the right i18n library, translates, then reviews the diff and reverts anything that isn't UI (prompts, state values, font names). Runs your typecheck, lint and tests. |
| [**sync-translations**](skills/sync-translations/SKILL.md) | *"Some pages still show English"*, *"Add Spanish"* | Finds missing keys, broken `{placeholders}` and bad ICU plurals, fills only the gaps or adds a language. Keeps every existing translation and changes no code. |
| [**find-hardcoded-strings**](skills/find-hardcoded-strings/SKILL.md) | *"How much work would i18n be?"* | Read-only audit: every hardcoded string by file and line, and which ones shouldn't be translated. |
| [**pseudo-localize**](skills/pseudo-localize/SKILL.md) | *"Test my UI with long text"* | Offline, no key: fake accented text (`⟦Sávé⟧`) shows missed strings, truncation and overflow before you pay for translations. |
| [**translate-docs**](skills/translate-docs/SKILL.md) | *"Translate our docs into Japanese"* | Markdown and MDX with code blocks, links, frontmatter and components intact. Fumadocs, Docusaurus, Starlight and more. |
| [**i18n-ci-check**](skills/i18n-ci-check/SKILL.md) | *"Fail CI when translations are missing"* | A GitHub Actions step (or any CI) that blocks missing keys and broken placeholders. No account or key needed. |
| [**auto-translate-on-push**](skills/auto-translate-on-push/SKILL.md) | *"Keep translations in sync automatically"* | Sets up the Babelize GitHub App: every push gets its new strings translated in one pull request that stays updated. Free plan included. |

Supports React, Next.js, Vue, React Native / Expo, Django and static HTML. Translations come from your own LLM key (Anthropic, OpenAI, Gemini, Groq, OpenRouter), Babelize's free hosted tier, or offline pseudo-localization.

## Why a skill, not just a prompt

Asking an agent to "add i18n" works once, for the files it happens to open. These skills drive the open-source [Babelize CLI](https://www.npmjs.com/package/babelize) instead, so:

- **Every file is covered**, with stable keys that are reused on the next run.
- **Re-runs are incremental**: a lockfile means only new or changed strings are translated (and paid for).
- **Existing translations are kept**, including the ones a human fixed.
- **The agent still reviews the result.** Extraction is pattern-based, so the skills tell the agent exactly what to check: LLM prompts, values the code compares, brand names, plurals, page metadata, `<html lang>`.

## Try it without a key

```text
"Show me what localizing this app into French would look like, offline"
```

The agent runs a pseudo-localization pass: you see every wrapped string and the i18n wiring in `git diff`, and nothing leaves your machine.

## Links

- Babelize: https://babelize.co
- CLI: `npx babelize --help` · [npm](https://www.npmjs.com/package/babelize)
- MCP server for agents: [`@babelize/mcp`](https://www.npmjs.com/package/@babelize/mcp)
- Issues and suggestions: [open an issue](https://github.com/babelize/skills/issues)

## License

MIT
