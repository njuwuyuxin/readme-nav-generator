---
name: readme-nav-generator
description: Generate layered README navigation files for a codebase by guiding the host agent to recursively inspect code and write one README.md per directory, including directory purpose, file summaries, and child-directory navigation. Use when users request AI-friendly project maps without external API setup, when reducing retrieval scope/token usage for large repositories, when users want to analyze only a manually specified subdirectory, or when users need Chinese navigation docs.
---

# Readme Nav Generator

Use this skill to generate a hierarchical README navigation layer through host-agent reasoning only. Do not call external LLM APIs from scripts.

## Execution mode

- Use local file tools to inspect code (`rg --files`, `Get-ChildItem`, `Get-Content`).
- Analyze code semantically from structure and behavior (symbols, imports, call paths, data flow), not only comments.
- Write or update `README.md` in each directory recursively.
- Keep output concise and navigation-oriented.
- Respect user-defined scope: analyze only the specified directory when provided.

## Scope selection

- If user provides a path (for example `src/services`), treat that path as the analysis root.
- If user asks for partial analysis but path is ambiguous, infer the most likely path from project structure and state it.
- If user does not specify scope, default to repository root.
- Never expand scope outside the chosen root unless user explicitly requests it.
- If the requested path does not exist, list a small set of closest candidate directories and ask the user to confirm one before generating files.

## Required workflow

1. Confirm target root (user-specified subdirectory or repo root), overwrite behavior, and excluded directories (for example `.git`, `node_modules`, `dist`, build caches).
2. Traverse directories recursively and gather code files per directory.
3. Summarize each file from code semantics:
   - Identify primary responsibility.
   - Identify important exported/entry symbols.
   - Identify key dependencies or integration points.
4. Summarize each directory from:
   - Direct files in that directory.
   - Child directory responsibilities.
5. Write `README.md` in every directory using the template below.
6. Regenerate from leaves upward if summaries drift after edits.

## Example user prompts

- `Use $readme-nav-generator to analyze only src/api and generate layered README.md files.`
- `Use $readme-nav-generator on backend/modules/auth only; do not touch other folders.`

## Output format

Use this structure for each generated `README.md`:

```md
# <relative/path>

## Directory purpose
<1-3 sentences about what this directory does in the system>

## File overview
- `<fileA.ext>`: <concise responsibility summary>
- `<fileB.ext>`: <concise responsibility summary>

## Subdirectory navigation
- `<childA>/`: <what it does>. See `<childA>/README.md`.
- `<childB>/`: <what it does>. See `<childB>/README.md`.

## Notes
- Keep this file concise and update when major code behavior changes.
```

## Quality rules

- Prefer behavior-based summaries over filename guesses.
- Avoid copying large code blocks into README files.
- Keep each file summary to one short line unless complexity requires two.
- Keep directory purpose short and specific.
- Preserve existing human-written details when compatible with current code.

## Language rule

- Output navigation README content in Chinese by default.
- Keep code identifiers, filenames, paths, API names, and symbols in their original form.
