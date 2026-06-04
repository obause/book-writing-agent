# AGENTS.md

## Project purpose

This repository contains `bookforge`, a local technical-book authoring system.

The goal is to help write a Fachbuch about Data Vault, AI, Data Warehousing, and Automation by using:

* LangGraph for stateful authoring workflows
* Quarto for book rendering
* Markdown/QMD as manuscript format
* YAML/JSON as machine-readable book state
* RAG for source retrieval
* structured review and revision workflows

The system should produce controlled, reviewable drafts. It must not be designed as an uncontrolled one-click book generator.

## Engineering principles

* Keep changes small and reviewable.
* Prefer simple, explicit code over clever abstractions.
* Use typed Python.
* Use Pydantic models for structured data.
* Use Typer for CLI commands.
* Use pytest for tests.
* Use ruff for linting and formatting.
* Keep LLM provider code behind interfaces.
* Do not require real API keys for tests.
* Do not commit secrets.
* Do not add a web UI unless explicitly requested.
* Do not add databases until the YAML/file-based MVP works.

## Domain principles

This is for a technical book, not fiction.

Prioritize:

* technical correctness
* terminology consistency
* citations and source traceability
* claim extraction and claim checking
* reproducible diagrams
* code examples
* structured review workflows
* human-in-the-loop approval

Avoid:

* hype-driven AI claims
* unsupported technical statements
* fake citations
* hallucinated sources
* uncontrolled autonomous generation

## Style conventions

* Use clear, practical technical language.
* Prefer explicit file formats and deterministic workflows.
* SQL examples must be lowercase.
* Generated manuscript files should be Quarto-compatible `.qmd`.
* Diagrams should preferably be Mermaid, PlantUML, Graphviz, SVG, or another versionable text-based format.

## Expected repository layout

Use a `src/bookforge` Python package layout.

Important areas:

* `src/bookforge/models` for Pydantic models
* `src/bookforge/storage` for file/YAML persistence
* `src/bookforge/workflows` for LangGraph workflows
* `src/bookforge/agents` for LLM-backed task modules
* `src/bookforge/rag` for retrieval
* `src/bookforge/quarto` for Quarto scaffolding and rendering
* `tests` for pytest tests
* `docs` for project documentation
* `examples/data-vault-ai-book` for a sample book project

## Done criteria for implementation tasks

A task is done only when:

* relevant tests are added or updated
* `uv run pytest` passes
* `uv run ruff check .` passes
* `uv run ruff format --check .` passes
* documentation is updated if behavior changed
* the implementation remains usable from the CLI
