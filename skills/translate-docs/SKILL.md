---
name: translate-docs
description: Translate Markdown and MDX documentation, guides and blog posts into other languages while keeping code blocks, links, frontmatter keys, MDX components and heading anchors intact. Writes each translation next to its page (page.ja.mdx) or into a per-language folder, and only re-translates pages that changed. Use when the user asks to translate docs, a docs site (Fumadocs, Docusaurus, Nextra, Astro Starlight), a README or MDX content.
---

# Translate docs

Uses the Babelize CLI (`npx babelize`, open source), which translates Markdown/MDX with the same glossary and tone as an app's strings. Needs Node 18+ and a git repo with a clean working tree.

## Configure once

Point it at the pages in `.babelize/config.json`:

```json
{ "docs": { "include": ["content/docs/**"] } }
```

Options inside `docs`:

| Option | Values | Use for |
|---|---|---|
| `include` / `exclude` | globs from the project root | which pages to translate |
| `layout` | `"dot"` (default): `install.mdx` → `install.ja.mdx` next to it. `"dir"`: `<root>/ja/install.mdx` | match how the docs framework finds translations |
| `root` | e.g. `"content/docs"` | the folder `dir` layout puts language folders under |
| `anchors` | `true` (default) / `false` | keeps `#anchor` links working by pinning each translated heading's id as `## Título [#title]` |

Pick them for the user's docs framework:

- **Fumadocs:** defaults work (`dot` layout and `[#id]` anchors are its own conventions).
- **Docusaurus, Nextra, Astro Starlight and others:** set `"anchors": false`, because `[#id]` would show as text. Check where the framework expects translated pages (Docusaurus: `i18n/<locale>/docusaurus-plugin-content-docs/current/`; Starlight: `src/content/docs/<locale>/`) and use `dir` layout with `root`, or move the files after the run. Tell the user if the framework's own i18n config also needs the locale added.

## Translate

```bash
npx babelize translate --to ja,de --provider <name> --docs-only --yes
```

Without `--docs-only` it also translates the app's strings, in the same run. A docs lockfile (`.babelize/local/docs-lock.json`) means later runs only redo changed paragraphs. Commit it.

Provider: long pages are much faster with the user's own key (`anthropic`, `openai`, `gemini`, `groq`, `openrouter` via `<NAME>_API_KEY`) than with Babelize's free hosted tier (`npx babelize auth login`, then `babelize`). Use `pseudo` to check the setup offline, then discard that output. Ask which; never ask for a key in chat.

Names that must stay as they are (product, CLI, commands) go in the glossary: `"glossary": { "neverTranslate": ["Acme CLI"] }`.

## Review

Spot-check a few translated pages:

- Fenced code blocks and inline code are unchanged.
- Frontmatter: keys unchanged; only text values like `title` and `description` translated; data values like `slug` untouched.
- MDX imports, components and their props intact; links and their `#anchors` intact.

Build or run the docs site if it has a build step, and open one translated page. Don't commit unless the user asks.
