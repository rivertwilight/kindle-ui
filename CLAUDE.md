# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kindle-UI is a React component library that recreates the Kindle e-reader interface in the browser. It provides 11+ interactive components styled to match the Kindle OS aesthetic, with support for dark mode and Kindle device frames (Oasis, Paperwhite, Voyage).

## Repository Structure

This is a **pnpm workspace monorepo** with the following packages:

- `packages/kindle-ui` - Main component library (@kindle-ui/core on npm)
- `packages/kindle-fonts` - Kindle-specific fonts (Bookerly, Amazon Ember)
- `docs/` - Next.js documentation site with component examples

## Build System

### Technology Stack
- **Build Tool**: Rollup with Turbo for caching
- **Language**: TypeScript with React
- **Styling**: styled-components
- **Test Framework**: Mocha with JSDOM
- **Package Manager**: pnpm (required, version 10.17.1+)

### Build Commands

```bash
# Build all packages (recommended)
pnpm build

# Build only library packages (skip docs)
pnpm build:packages

# Build only documentation
pnpm build:docs

# Development mode with watch
pnpm dev
```

### Output Formats
The library builds to **dual format** outputs:
- `dist/index.cjs.js` - CommonJS
- `dist/index.esm.js` - ES Modules
- `dist/index.d.ts` - TypeScript definitions

## Testing

```bash
# Run unit tests
pnpm test:unit
```

Tests are configured via `.mocharc.js` and use:
- Mocha test runner
- Babel transpilation for TypeScript/JSX
- JSDOM for DOM environment
- Tests should be named `*.test.{js,ts,tsx}`

## Development Workflow

### Making Changes to Components

1. Component source files are in `packages/kindle-ui/src/components/`
2. Each component exports from its own `index.tsx`
3. All public exports must be added to `packages/kindle-ui/src/index.ts`
4. After building, the postbuild script copies README.md to the package

### Working with the Docs Site

The docs site uses **Next.js 15** with Turbopack:

```bash
cd docs
pnpm dev      # Start dev server with turbopack
pnpm build    # Build for production
```

Example components are in `docs/src/components/examples/`.

## Architecture

### Theme System

The library uses a centralized theme system (`packages/kindle-ui/src/theme/index.ts`):
- Defines color constants for light/dark modes
- Exports `theme` object with `light` and `dark` variants
- Colors mimic e-ink display aesthetics (black, white, greys)

### Container Component Pattern

The `Container` component is the **root wrapper** that provides:
- Theme context (dark/light mode)
- CSS variables (`--text-color`, `--bg-color`, `--text-color-secondary`)
- Optional device frame rendering
- Font family defaults (Amazon Ember, Bookerly)

All Kindle-UI apps should wrap content in `<Container>` to enable theming.

### Navbar Architecture

The Navbar is a **composite component** consisting of:
- `StatuBar` - Shows battery, cellular, wifi, device name
- `ActionBar` - Contains action buttons and search
  - `ActionGroup` - Groups related actions
  - `ActionItem` - Individual buttons
  - `ActionBarSpace` - Flexible spacer
  - `ActionBarMenu` - Dropdown menu
  - `SearchBar` - Search input

### SVG Icons

Icons are imported as React components via `@svgr/rollup`:
```tsx
export { default as HomeOutlineIcon } from "./icons/home-outline.svg";
```

## Key Dependencies

### Core Library (packages/kindle-ui)
- `styled-components` ^5.3.11 - CSS-in-JS styling
- `react` & `react-dom` >=17.0.2 (peer dependencies)

### Build Dependencies
- `@rollup/plugin-babel` - Babel transpilation
- `@rollup/plugin-typescript` - TypeScript support
- `@svgr/rollup` - SVG to React component transformation
- `rollup-plugin-url` - Font file bundling (inline fonts as base64)
- `rollup-plugin-delete` - Clean dist folder

## Publishing

```bash
# Publish core library
pnpm publish:core

# Publish fonts package
pnpm publish:fonts
```

The `prepublishOnly` script in root package.json:
1. Copies README.md to packages/kindle-ui/
2. Runs full build
3. Builds documentation

Published packages are scoped under `@kindle-ui/` on npm.

## Important Notes

### Styled Components Theming
Components use styled-components with theme props passed directly (not ThemeProvider). The `Container` component sets CSS variables that styled components reference.

### Component Prop Patterns
Most interactive components support:
- Standard React props (`onClick`, `className`, etc.)
- Polymorphic `component` prop for rendering as different elements
- Example: `<ListItem component="a" href="...">` or `<ListItem component={Link} to="...">`

### TypeScript Configuration
- Uses Babel preset-typescript (not tsc) for compilation
- Type checking happens via `@rollup/plugin-typescript`
- Component interfaces are exported with `I` prefix (e.g., `IContainer`)

### Font Integration
Fonts are bundled using `rollup-plugin-url` with `limit: Infinity` to ensure fonts are inlined as base64 rather than copied as separate files. The optional `kindle-fonts` package provides CSS imports for authentic Kindle typography.
