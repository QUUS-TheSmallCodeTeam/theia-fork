# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Eclipse Theia repository - an extensible framework for developing full-fledged multi-language Cloud & Desktop IDEs and tools. Theia is built with TypeScript and supports both browser-based and Electron-based applications.

## Key Architecture

### Platform Structure
Code is organized by platform compatibility:
- `common/*`: Basic JavaScript APIs, runs everywhere
- `browser/*`: Browser DOM APIs (may use common)
- `node/*`: Node.js APIs (may use common)
- `electron-node/*`: Electron + Node.js APIs (may use common, node)
- `electron-browser/*`: Electron renderer APIs (may use common, browser)
- `electron-main/*`: Electron main process APIs (may use electron-node, common, node)

### Repository Structure
- `packages/`: Core runtime packages and extensions (~80 packages)
- `dev-packages/`: Development tools and utilities
- `examples/`: Browser and Electron example applications
- `doc/`: Documentation
- `scripts/`: Build and utility scripts

### Core Packages
- `@theia/core`: Foundation framework with DI, widgets, commands
- `@theia/monaco`: Monaco editor integration
- `@theia/filesystem`: File system abstraction
- `@theia/editor`: Editor framework
- `@theia/debug`: Debug support
- `@theia/plugin-ext`: VS Code extension compatibility
- `@theia/ai-*`: AI integration packages (Anthropic, OpenAI, etc.)

## Development Commands

### Build Commands
```bash
npm install                    # Install dependencies and bootstrap
npm run compile               # Compile TypeScript packages
npm run build                 # Full build (compile + applications)
npm run build:browser         # Build browser example
npm run build:electron        # Build Electron example
npm run all                   # Install + lint + build everything
```

### Development Server
```bash
npm run start:browser         # Start browser example (http://localhost:3000)
npm run start:electron        # Start Electron example
npm run watch:browser         # Watch browser example
npm run watch:electron        # Watch Electron example
npm run watch                 # Watch all packages
```

### Testing
```bash
npm run test                  # Run all tests
npm run test:theia           # Run Theia package tests only
npm run test:browser         # Run browser example tests
npm run test:electron        # Run Electron example tests
npx lerna run test --scope @theia/package-name  # Test specific package
```

### Linting and Quality
```bash
npm run lint                  # Lint all packages
npm run lint:fix             # Auto-fix linting issues
npx lerna run lint --scope @theia/package-name  # Lint specific package
```

### Package Management
```bash
npx lerna run compile --scope @theia/package-name     # Compile specific package
npx lerna run watch --scope @theia/package-name       # Watch specific package
npm run download:plugins      # Download VS Code plugins for examples
```

## Coding Guidelines

### Code Style
- Use 4 spaces for indentation
- Use single quotes for strings (except template literals)
- Use PascalCase for types and enums
- Use camelCase for functions, methods, properties, variables
- Use lower-case, dash-separated file names
- Always declare explicit return types

### TypeScript Patterns
- Use classes instead of interfaces + symbols when possible
- Use property injection over constructor injection
- Use `@postConstruct()` for initialization
- Always add `inSingletonScope()` for singletons
- Use `undefined` instead of `null`
- Don't export functions unless necessary (convert to class methods)

### Internationalization
- Always localize user-facing text with `nls.localize(key, defaultValue, ...args)`
- Use `nls.localizeByDefault(defaultValue)` for VS Code compatible strings
- Pass parameters as args, not string interpolation

### URI/Path Handling
- Always pass URIs between frontend/backend, never paths
- Use explicit URI schemes
- Frontend: Use `FileService.fsPath` and `Path` API
- Backend: Use `FileUri.fsPath` and Node.js `fs`/`path` APIs
- Use `LabelProvider.getName/getLongName/getIcon` for display

### Dependency Injection
- Use `ContributionProvider` instead of `@multiInject`
- Prefer property injection with `@inject()`
- Use `@postConstruct()` for initialization logic
- Always bind with `inSingletonScope()` for singletons

### Testing Structure
- `*.spec.ts`: Unit tests (published)
- `*.slow-spec.ts`: Integration tests (unpublished)
- `*.ui-spec.ts`: UI tests (unpublished)
- Place test helpers in `test/` subdirectories

## Extension Development

### Creating Extensions
Extensions are defined in `package.json` via `theiaExtensions` field with module paths for different platforms (frontend, backend, frontendElectron, backendElectron).

### VS Code Compatibility
Use `@theia/plugin-ext` for VS Code extension support. Mark incomplete implementations with `@stubbed` tag.

### Monaco Integration
Use `@monaco-uplift` tag for features requiring newer Monaco versions.

## AI Integration

This repository includes extensive AI tooling in packages prefixed with `ai-`:
- Core AI framework (`@theia/ai-core`, `@theia/ai-core-ui`)
- Provider integrations (Anthropic, OpenAI, Google, Ollama, etc.)
- AI features (chat, code completion, terminal integration)
- MCP (Model Context Protocol) support

## Debugging

### Browser Example
- Backend: Use VS Code launch config "Launch Browser Backend"
- Frontend: Start backend, then use dev tools or "Launch Browser Frontend" config

### Electron Example  
- Backend: Use "Launch Electron Backend" config
- Frontend: Use "Attach to Electron Frontend" or Electron dev tools

### Plugin Host
- Add `--hosted-plugin-inspect=9339` to backend
- Use "Attach to Plugin Host" launch config

## Performance Notes

- Use `console` instead of `ILogger` for top-level logging
- Avoid binding functions in React event handlers
- Use arrow function class properties for React event handlers
- Be mindful of file watcher limits on Linux (increase `fs.inotify.max_user_watches`)

## EG-Desk Project

This fork is being used to develop **eg-desk_taehwa**, a custom application built on the Theia framework.

### Project Structure
- `eg-desk_taehwa/`: Custom application directory (separate from examples)
- Uses Theia's AI integration packages for intelligent development assistance
- Built as a standalone application leveraging the monorepo's shared packages

### EG-Desk Development
- The AI agent will analyze the Theia codebase structure to provide insights on available features
- Focuses on leveraging Theia's extensive AI tooling (`@theia/ai-*` packages)
- Independent build system separate from main Theia examples
- Can access all Theia packages via the monorepo's shared node_modules
