# Repository Guidelines

## Agent Roles

Use `ba`, `dev`, `qa`, and `dockeeper` as appropriate; see [workflow](docs/agent-workflow.md). Definitions live in `.codex/agents/`.

## Project Structure & Module Organization

This project will be developed in Rust and run in Docker. Application code, Cargo manifests, and Docker configuration have not been added yet; the existing `.gitignore` is a generic Visual Studio template.

Place application code in `src/`, integration tests in `tests/`, and supporting documentation in `docs/`. Use `src/main.rs` for the executable entry point and `src/lib.rs` for shared library code when needed. Keep `Cargo.toml`, `Cargo.lock`, `Dockerfile`, and `.dockerignore` at the repository root.

## Build, Test, and Development Commands

Once `Cargo.toml` and Docker configuration exist, use:

- `cargo build --locked` - compile with committed dependency versions.
- `cargo run` - run the application locally.
- `cargo test --locked` - run unit, integration, and documentation tests.
- `cargo fmt --check` - check formatting; use `cargo fmt` to apply it.
- `cargo clippy --all-targets -- -D warnings` - reject lint warnings.
- `docker build -t maildev .` - build the container image.
- `docker run --rm --env-file .env maildev` - run with local configuration, when required.

Document the Rust toolchain, runtime variables, and required port mappings in `README.md` when implemented.

## Coding Style & Naming Conventions

Use rustfmt defaults with four-space indentation. Name modules, files, and functions in `snake_case`, types and traits in `PascalCase`, and constants in `SCREAMING_SNAKE_CASE`. Keep modules focused and propagate recoverable errors with `Result` and `?`. Avoid `unwrap()` and `expect()` in production paths unless the invariant is documented.

## Testing Guidelines

Use Rust's built-in test framework: unit tests in module-local `#[cfg(test)]` blocks and integration tests in `tests/`. Name tests after observable behavior, such as `rejects_invalid_configuration`. Add regression tests for bug fixes. Keep default tests independent of external services; document separate container-dependent checks. No numerical coverage threshold is established.

## Commit & Pull Request Guidelines

The repository has no commits yet. Use concise, imperative commit subjects, such as `Add Docker build for Rust service`.

Pull requests should describe the change, its purpose, and validation performed. Link relevant issues and include screenshots for visible UI changes. Identify any setup or configuration changes.

## Security & Configuration

Commit `Cargo.lock` for reproducible application builds. Exclude `target/`, secrets, and local `.env` files from Git and Docker build contexts. Provide `.env.example` with placeholders. Use multi-stage Docker builds and a non-root runtime user; never bake credentials into images.
