# Repository Guidelines

## Project Structure & Module Organization
- `packages/` hosts publishable Theia extensions such as `core`, `editor`, and AI integrations; each package follows `src/{browser,node,common}` pattern with adjacent `*.spec.ts` tests.
- `dev-packages/` provides tooling modules and CLIs used only for development.
- `examples/` offers runnable applications (`browser`, `electron`, `browser-only`, `playwright`) and API samples; use them to validate features end-to-end.
- `doc/` and `misc/` capture architectural notes and supporting assets; update docs alongside significant code changes.

## Build, Test, and Development Commands
- `npm install` installs workspace dependencies (requires Node >= 20).
- `npm run compile` compiles all packages via Lerna.
- `npm run build:browser` / `npm run build:electron` create runnable IDE bundles.
- `npm run start:browser` (or `start:electron`) launches a development instance.
- `npm run lint` executes ESLint/TSLint suites; `npm run lint:fix` applies safe fixes.
- `npm run test`, `npm run test:theia`, and `npm run test:browser` cover unit, integration, and example-app tests respectively.

## Coding Style & Naming Conventions
- Follow `.editorconfig`: LF endings, spaces, 4-space indent for TS/JS/MD, 2 for JSON/YAML.
- Prefer TypeScript; organize code into `src/browser`, `src/node`, and `src/common` directories.
- Name packages `@theia/<feature>` (kebab-case); classes PascalCase; services/interfaces prefixed with `I`.
- Run `npm run lint` before pushing; resolve formatting and lint warnings instead of bypassing them.

## Testing Guidelines
- Write unit specs in `*.spec.ts` using Mocha + Chai assertion helpers.
- Place integration tests in `examples/api-tests` or relevant example apps; reuse the Playwright suite with `npm run test:playwright`.
- Ensure new features are covered by at least one automated test; use `nyc` locally when touching critical paths to watch coverage.
- Keep test fixtures under `packages/<feature>/src/tests` or `misc/` to avoid polluting runtime assets.

## Commit & Pull Request Guidelines
- Use imperative, scoped commit subjects mirroring the existing history (for example `fix(chat): guard suggestion panel` or `Add terminal metrics (#16229)`), and append a `Signed-off-by` line to satisfy the Eclipse DCO.
- Reference issues in commit bodies, describe user impact, and note the verification commands you ran.
- PRs must pass CI, include screenshots or recordings for UI changes, and update changelogs or docs when behavior shifts.
- Link to supporting design docs (such as `doc/pull-requests.md`) when workflow deviations need justification.

## Security & Configuration Tips
- Report vulnerabilities privately to the Eclipse Security Team per `SECURITY.md`; never disclose them in public issues or PRs.
- Keep bundled plugins current with `theia download:plugins` and avoid committing credentials; store secrets in environment variables or developer-specific `.theia` settings.
