# Repository Guidelines

## Project Structure & Module Organization
- `packages/cli/`: CLI that installs and manages components; built with `tsup`.
- `packages/registry/`: shared registry types/schemas/utilities; published package.
- `docs/`: SvelteKit documentation site and content build pipeline.
- `registry-template/`: SvelteKit template used to generate registry assets.
- `repro/`: minimal repro template for issue reports.
- Root configs: `pnpm-workspace.yaml`, `eslint.config.js`, `.prettierrc`.

## Build, Test, and Development Commands
- `pnpm dev`: run the docs site in dev mode (SvelteKit).
- `pnpm build`: build the docs site (default root build).
- `pnpm build:cli`: build all packages under `packages/`.
- `pnpm check`: run `svelte-check` and TypeScript checks across packages.
- `pnpm lint` / `pnpm format`: run ESLint + Prettier or write formatting.
- `pnpm -F shadcn-svelte test`: run CLI tests (includes registry-template build).
- `pnpm -F @shadcn-svelte/registry test`: run registry tests (Vitest).

## Coding Style & Naming Conventions
- Use `pnpm` (enforced by `preinstall`); Node >= 20 and pnpm >= 9 required.
- Formatting is Prettier-driven: tabs, width 4, double quotes, trailing commas; Markdown uses 2-space indentation.
- ESLint is configured for TS/Svelte; follow existing Svelte and TypeScript patterns in each package.
- Keep file and directory naming consistent with existing package scopes (e.g., `packages/cli`, `packages/registry`).

## Testing Guidelines
- Tests are run with Vitest in `packages/cli` and `packages/registry`.
- Prefer colocated tests within the package they validate.
- Run relevant tests per package before opening a PR; there is no stated coverage target.

## Commit & Pull Request Guidelines
- Commit messages follow Conventional Commits (e.g., `fix: ...`, `docs: ...`, `chore(deps-dev): ...`).
- For substantial changes, start a discussion first (see `CONTRIBUTING.md`).
- PRs should include a clear description and link issues/discussions; add screenshots for UI/doc changes when relevant.

## Contributor Notes
- For bug reports, consider using the `repro/` template to provide a minimal reproduction.
