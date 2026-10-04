# Repository Guidelines

## Project Structure & Module Organization

This repository documents the evolution of AdScale’s advertising platform; it does not contain an executable application.

- `README.md`: brief project introduction.
- `docs/context.md`: business context, current architecture, constraints, and delivery goals.
- `Task1/task.md`: assignment requirements and the existing C4 Container diagram in a PlantUML block.
- Planned Task 1 deliverables: `Task1/README.md`, `AS-IS.md`, `Drivers.md`, `TO-BE.md`, `Diagrams/`, and `adr/`, all under `Task1/`.
- `Task2/` and `Task3/` are prescribed by the project brief but have not been created. Keep each assignment’s artifacts in its own directory.

## Build, Test, and Development Commands

There is no configured build, application runtime, dependency manifest, or automated test suite. Run these checks from the repository root:

- `git status --short`: review changed and untracked files.
- `git diff --check`: detect whitespace errors in unstaged changes.
- `git diff --cached --check`: check staged changes before committing.
- `git diff -- Task1/ docs/ README.md`: review documentation changes.

Preview Markdown locally. Render PlantUML diagrams with a compatible renderer, or open Draw.io sources in Draw.io; no renderer is bundled.

## Documentation Style & Naming Conventions

Use concise Russian prose to match the existing project documents. Keep technical identifiers consistent across prose, diagrams, and ADRs. Use Markdown headings, fenced code blocks with language tags, and relative links. Use spaces rather than tabs; indent PlantUML container contents by four spaces, matching the existing diagram.

Preserve assignment filenames and casing, including `AS-IS.md`, `TO-BE.md`, and `Diagrams/`. Name decision records with sequential identifiers, for example `adr/ADR-001-evolution-strategy.md`.

## Validation Guidelines

No testing framework or coverage threshold is configured. Validate document links, diagram rendering, component names, and agreement between diagrams and ADRs. Distinguish current measurements, three-month targets, and one-year goals. Flag conflicting latency requirements in the brief and document assumptions explicitly.

## Commit & Pull Request Guidelines

Git history contains only `Initial commit`, so no established message convention exists. Use short, descriptive imperative subjects, such as `Document Task1 architecture drivers`, and keep commits focused.

Submit work through a pull request as required by the brief. Describe affected assignments, key decisions, and validation performed; link related issues when available. Include diagram previews when useful for review. Exclude local IDE metadata and generated logs.
