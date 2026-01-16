# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install                     # Install dependencies (requires Node >=22)
npx quartz build --serve        # Build and serve locally with hot reload
npx quartz build -d docs        # Build docs directory
npm run check                   # Type check + format check
npm run format                  # Auto-format with Prettier
npm run test                    # Run tests (tsx --test)
npx quartz sync --pull          # Pull updates from upstream Quartz
```

## Architecture

Quartz is a static site generator that transforms Markdown content into a website. Content flows through a three-stage pipeline:

```
Markdown Files → [PARSE] → [FILTER] → [EMIT] → Static HTML
```

### Plugin System

Three plugin types hook into this pipeline:

**Transformers** (`quartz/plugins/transformers/`) - Modify content during parsing
- `textTransform()` - Pre-process raw text
- `markdownPlugins()` - Remark plugins for Markdown AST
- `htmlPlugins()` - Rehype plugins for HTML AST

**Filters** (`quartz/plugins/filters/`) - Control which content gets published
- `shouldPublish()` - Return boolean to include/exclude content

**Emitters** (`quartz/plugins/emitters/`) - Generate output files
- `emit()` - Main output generation
- `partialEmit()` - Incremental rebuilds in watch mode

### Component System

Components are Preact components with optional CSS and client-side scripts:

```typescript
type QuartzComponent = ComponentType<QuartzComponentProps> & {
  css?: string              // Inlined into page
  beforeDOMLoaded?: string  // Runs in <head>
  afterDOMLoaded?: string   // Runs after DOMContentLoaded
}
```

Components receive: `ctx` (build context), `fileData` (page metadata), `cfg` (config), `tree` (HTML AST), `allFiles` (for cross-references).

### Configuration

- `quartz.config.ts` - Global settings and plugin configuration
- `quartz.layout.ts` - Component layout (what appears in header, sidebar, footer)

### Build Pipeline

The build (`quartz/build.ts`) orchestrates:
1. Glob content files, respecting `ignorePatterns`
2. Parse in parallel using worker threads (chunks of 128 files)
3. Filter sequentially through each filter plugin
4. Emit in parallel from all emitter plugins
5. In watch mode, track changes and use `partialEmit()` for efficiency

### Key Types

- `BuildCtx` - Build context passed to plugins (argv, cfg, allSlugs, allFiles)
- `QuartzPluginData` - VFile data (slug, frontmatter, filePath)
- `ProcessedContent` - Tuple of [HtmlRoot, VFile] after parsing
