Milestone 0 — Project foundation

Goal: create the repo, package, CLI, models, tests, and Quarto scaffold.

Commands you want after this milestone:

uv sync
uv run pytest
uv run bookforge init examples/data-vault-ai-book
uv run bookforge validate examples/data-vault-ai-book
uv run bookforge render examples/data-vault-ai-book

No LangGraph yet. No LLMs yet.

This is important because Codex performs much better when the project has tests, structure, and commands.

Milestone 1 — Quarto book scaffold

Build proper Quarto output.

Generated project:

examples/data-vault-ai-book/
  _quarto.yml
  index.qmd
  references.bib
  chapters/
    ch_01.qmd
  knowledge/
    claims.yml
    glossary.yml
    figures.yml
    section_summaries.yml
  generated/
    drafts/
    reviews/
    revised/

Minimum _quarto.yml:

project:
  type: book

book:
  title: "Data Vault, AI, and Automation"
  author: "Ole Bause"
  chapters:
    - index.qmd
    - chapters/ch_01.qmd

bibliography: references.bib

format:
  html:
    toc: true
    number-sections: true
  pdf:
    toc: true
    number-sections: true
  docx: default

At this point, the tool should be able to create a renderable empty book.

Milestone 2 — LangGraph workflow skeleton

Now add LangGraph, but still without real LLM calls.

Build a section workflow with mock agents:

load_section_plan
→ mock_retrieve_sources
→ mock_write_draft
→ mock_extract_claims
→ mock_review
→ mock_revise
→ save_outputs

Why mock first? Because you want to test the workflow deterministically before spending money on LLM calls.

LangGraph is deliberately low-level and focused on orchestration, durable execution, streaming, human-in-the-loop, and persistence, so you want your graph state and node boundaries clean before connecting models.

Output files:

generated/drafts/ch_01_s01.md
generated/reviews/ch_01_s01.review.yml
generated/revised/ch_01_s01.md
knowledge/claims.yml
knowledge/section_summaries.yml

CLI:

uv run bookforge draft-section examples/data-vault-ai-book ch_01_s01 --mock
Milestone 3 — LLM provider interface

Add real model calls behind an interface.

Suggested abstraction:

class LLMProvider(Protocol):
    def complete_text(self, prompt: str) -> str: ...

    def complete_structured(
        self,
        prompt: str,
        schema: type[BaseModel],
    ) -> BaseModel: ...

Do not scatter OpenAI/Anthropic/Gemini calls throughout the codebase.

Keep this separation:

agents/
  writer.py
  reviewer.py

llm/
  provider.py
  openai_provider.py
  mock_provider.py

CLI options:

uv run bookforge draft-section examples/data-vault-ai-book ch_01_s01 --provider openai
uv run bookforge draft-section examples/data-vault-ai-book ch_01_s01 --provider mock

Tests should use the mock provider only.

Milestone 4 — Prompt system

Add prompt templates as versioned files.

prompts/
  book_planner.md
  chapter_planner.md
  section_writer.md
  claim_extractor.md
  technical_reviewer.md
  redundancy_reviewer.md
  reviser.md
  memory_agent.md
  diagram_agent.md

Each prompt should have:

role
goal
input data
constraints
output schema
quality rules

For your book, the Writer prompt should include rules like:

- Do not invent sources.
- Do not overstate AI capabilities.
- Explain Data Vault terms precisely.
- Prefer practical examples.
- Mark unsupported claims explicitly.
- Keep the tone technical, clear, and non-hype.
Milestone 5 — Claim registry

This is the most important Fachbuch-specific feature.

Add commands:

bookforge extract-claims <project-dir> <section-id>
bookforge list-claims <project-dir>
bookforge validate-claims <project-dir>

Statuses:

draft
needs_source
verified
weakly_supported
author_experience
opinion
rejected

The Technical Reviewer should fail or warn if a section has many needs_source claims.

Milestone 6 — RAG/source ingestion

Start simple.

Support:

sources/notes/*.md
sources/web/*.md
sources/pdfs/*.pdf later

Do not start with complex PDF parsing. Start with Markdown notes.

Commands:

bookforge ingest-sources <project-dir>
bookforge search-sources <project-dir> "data vault automation"

Initial RAG can be:

local chunks as JSONL
embeddings later
simple keyword search first
vector search later

This keeps the MVP usable before adding complexity.

Milestone 7 — Diagram agent

Add a figure registry:

figures:
  - id: fig_metadata_driven_dbt_generation
    title: "Metadata-driven dbt generation"
    type: mermaid
    status: draft
    source_file: figures/fig_metadata_driven_dbt_generation.mmd
    used_in:
      - ch_04_s02

The Diagram Agent should generate Mermaid first.

Command:

bookforge suggest-figures <project-dir> ch_04
bookforge create-figure <project-dir> fig_metadata_driven_dbt_generation

Do not start with image generation. For a technical book, Mermaid/PlantUML/SVG is better.

Milestone 8 — Chapter assembly

Once sections exist, add:

bookforge assemble-chapter <project-dir> ch_03

This should:

collect revised sections
insert them into chapters/ch_03.qmd
include figure references
include citations
add chapter intro/conclusion if configured
preserve manual edits carefully

Important: avoid overwriting human-edited chapter files without backup/diff.

Milestone 9 — Quality reports

Add:

bookforge report <project-dir>

Report:

chapters planned: 12
sections planned: 84
sections drafted: 19
sections reviewed: 16
sections human-approved: 7
claims total: 213
claims verified: 87
claims needing source: 42
figures planned: 18
figures implemented: 5

This turns the book into a manageable production project.
