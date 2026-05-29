# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vite plugin that automatically injects `@reference` directives into CSS files to enable Tailwind's `@apply` directive. The plugin solves the problem where CSS files using `@apply` don't have access to Tailwind utilities without manual `@reference` imports.

**Key behavior**: The plugin intercepts CSS files during Vite's transform phase, detects `@apply` usage, and prepends the necessary `@reference` directive to the CSS file(s) containing Tailwind utilities. It works with plain CSS files by default and can be configured to support framework-scoped styles (Svelte, Vue, etc.) via the `include` option.

## Architecture

**Entry points:**

- `src/index.ts` - The plugin source code (exported as library)

**Build configuration:**

- The plugin is built as an ES module library via `tsdown`
- Output: `dist/index.mjs` and `dist/index.d.mts` (exported through `package.json`)

**Plugin flow:**

1. `configResolved` hook: Initializes the file filter using Vite's `createFilter` with include/exclude patterns
2. `transform` hook (enforce: "pre"): Intercepts files matching the filter (default: `/\.css$/`)
3. Checks if code contains `@apply` and isn't skipped by the `skip` callback
4. Inserts `@reference` directive(s) either at the top or after the last `@use` statement
5. Resolves the CSS file path relative to project root using `node:path`

**Key plugin configuration:**

- `cssFile`: Path(s) to Tailwind CSS file or a function returning them (default: `"tailwindcss"`)
- `include`: Files to transform (default: CSS files via `/\.css$/`)
- `exclude`: Files to skip
- `skip`: Custom function to skip transformation based on code/id

## Development Commands

```bash
# Build the library
bun run build

# Lint
bun run lint

# Format
bun run format
```

## Important Notes

- The plugin uses `enforce: "pre"` to run before other transforms
- Reference paths are resolved using `resolve(root, file)` from the Vite config root
- The plugin preserves existing `@use` statements by inserting references after them
- Transform returns `null` if file doesn't match filter or doesn't contain `@apply`
- Files already containing `@reference` or `@import "tailwindcss"` are skipped
- The `cssFile` parameter can be a string, array, or async function for dynamic resolution
