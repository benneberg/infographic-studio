Infographic Studio — Architecture

Status: Draft
Architecture version: 0.1

⸻

1. Architectural Goal

Infographic Studio should evolve from the current single-file Studio application into a deterministic visual generation system with multiple interfaces sharing one rendering core.

The central architectural principle is:

There must be exactly one authoritative rendering implementation.

Studio, CLI, and GitHub Action must ultimately invoke the same engine.

⸻

2. Current Architecture

The current application is a single-file, zero-dependency browser application.

It currently provides:

┌───────────────────────────────┐
│        Infographic Studio     │
│                               │
│  Projects                     │
│  Sources                      │
│  Settings                     │
│  Style                        │
│  Result                       │
└───────────────┬───────────────┘
                │
        ┌───────▼────────┐
        │ Project State  │
        │ localStorage   │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Source Ingest  │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Optional LLM   │
        │ Generation     │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Infographic    │
        │ Spec           │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ SVG Renderer   │
        └───────┬────────┘
                │
          ┌─────┴─────┐
          ▼           ▼
         SVG         PNG

The current repository implements this architecture primarily inside index.html.

The application already contains:

* project persistence
* source ingestion
* project options
* themes
* formats/aspects
* LLM providers
* structured infographic specifications
* deterministic SVG rendering
* SVG/PNG export

This existing implementation is the starting point for the migration.

⸻

3. Target Architecture

The target architecture separates the system into distinct layers.

                         ┌────────────────────┐
                         │    Data Sources    │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  Project / Data    │
                         │     Specification  │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │     Validator      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  Template Engine   │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Deterministic      │
                         │ Renderer            │
                         └─────────┬──────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                  SVG             PNG            PDF

The engine is then exposed through:

                    ┌────────────────────┐
                    │    Core Engine     │
                    └─────────┬──────────┘
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          Studio             CLI        GitHub Action

⸻

4. Architectural Layers

4.1 Project Layer

Defines the complete infographic project.

Responsibilities:

* project metadata
* data
* content
* template reference
* theme reference
* canvas
* configuration
* version information

It must be serializable.

⸻

4.2 Schema Layer

Defines valid structures for:

* projects
* templates
* themes
* data
* bindings
* renderer configuration

The schema should be versioned.

⸻

4.3 Validation Layer

Validates a project before rendering.

Validation should detect:

* missing fields
* invalid types
* unsupported template options
* missing assets
* invalid references
* incompatible versions
* layout constraint violations

Validation should be usable independently of rendering.

⸻

5. Template System

Templates define the semantic structure of an infographic.

A template should not directly depend on the Studio UI.

A template should be renderable headlessly.

Conceptually:

Template
├── metadata
├── canvas rules
├── sections
├── patterns
├── bindings
├── configuration
└── constraints

Templates should use semantic patterns rather than manually positioning every primitive.

⸻

6. Visual Primitives

Primitives are the lowest-level visual building blocks.

Initial examples:

Text
Rectangle
Line
Circle
Icon
Image
Number
Chart

Primitives should be as stateless and deterministic as possible.

⸻

7. Patterns

Patterns are reusable semantic compositions built from primitives.

Examples:

StatCard
MetricRow
ComparisonCard
TimelineStep
Callout
Badge
ProgressBar
RankingItem

Patterns provide a useful abstraction between low-level rendering and full infographic templates.

⸻

8. Templates

Templates compose patterns into complete infographic structures.

Examples:

MetricsSummary
Comparison
Timeline
Process
Ranking

Templates should primarily describe:

* structure
* available slots
* data bindings
* supported variants
* constraints

They should not contain application-specific state.

⸻

9. Themes

Themes define the visual design system.

A theme may contain:

Colors
Typography
Spacing
Border widths
Border radii
Shadows
Icon style
Chart palette
Backgrounds

Themes should be independent from templates wherever possible.

The same theme should be usable with multiple templates.

⸻

10. Data Binding

Templates should consume structured data through explicit bindings.

Conceptually:

Template slot
      │
      ▼
data.metrics[0].value

The renderer must not infer meaning from arbitrary strings.

Data transformations should be explicit and deterministic.

⸻

11. Rendering

The renderer converts a validated project into an intermediate visual representation and finally into an output format.

The preferred canonical output is SVG.

Conceptually:

Project
   ↓
Validation
   ↓
Template resolution
   ↓
Data binding
   ↓
Layout calculation
   ↓
Visual tree
   ↓
SVG renderer
   ↓
SVG

Rasterization to PNG should happen after SVG generation.

⸻

12. SVG as Canonical Representation

SVG is the preferred canonical output because it is:

* scalable
* inspectable
* text-based
* versionable
* web-compatible
* suitable for accessibility metadata

PNG should be treated as a derived artifact.

PDF may eventually be another derived output.

⸻

13. Layout Engine

The layout engine is responsible for deterministic placement and sizing.

It should use explicit constraints.

Examples:

minimum width
maximum width
minimum height
maximum height
padding
gap
alignment
text size range
item count limits

The layout engine must not depend on uncontrolled browser layout behavior.

⸻

14. Text Rendering

Text is a major determinism risk.

Browser text measurement may differ from headless rendering environments.

The architecture should therefore avoid making the browser’s layout engine the authoritative source of truth.

Potential strategies include:

* bundled fonts
* explicit font metrics
* deterministic text measurement
* SVG text with controlled fonts
* optional text-to-path conversion for strict rendering

The final implementation choice should be made after renderer extraction and testing.

⸻

15. Determinism

The rendering core must not use uncontrolled:

Math.random()
Date.now()
new Date()

for visual generation.

If deterministic variation is required:

project seed
      ↓
seeded PRNG
      ↓
reproducible variation

The seed must be part of the project specification.

⸻

16. External Resources

External resources should not be silently fetched during rendering.

Examples:

* fonts
* images
* icons
* remote data

A project should explicitly identify external dependencies.

For CI/reproducible builds, resources should preferably be:

* local
* bundled
* pinned
* content-addressed
* or otherwise versioned

⸻

17. Studio

Studio is the visual authoring environment.

It should interact with the core through project specifications rather than duplicating rendering logic.

Studio
 ├── Project editor
 ├── Data editor
 ├── Template browser
 ├── Theme editor
 ├── Preview
 ├── Validation
 └── Export

Studio may use browser-specific rendering for interactive preview, but the authoritative rendering implementation should remain shared with headless environments.

⸻

18. CLI

The CLI is the primary headless interface.

Initial commands:

infographic validate project.yml
infographic render project.yml

Potential future commands:

infographic preview project.yml
infographic inspect project.yml
infographic batch projects/
infographic diff before.svg after.svg
infographic test

The CLI must not implement its own rendering engine.

⸻

19. GitHub Action

The GitHub Action should be a thin wrapper around the CLI.

Conceptually:

GitHub Action
      │
      ▼
     CLI
      │
      ▼
 Core Engine

The Action should not contain business logic that belongs in the core.

Potential use cases:

* README images
* release graphics
* project metrics
* scheduled reports
* documentation generation
* pull request previews

⸻

20. Optional AI Layer

AI is an optional authoring mechanism.

It may convert:

source material
      ↓
AI
      ↓
structured project/data

The output must then pass through:

validation
      ↓
deterministic renderer

AI must never be required for rendering.

This allows the current LLM functionality to remain useful without making the product AI-dependent.

⸻

21. Source Ingestion

Source ingestion is separate from rendering.

Possible sources include:

* pasted text
* GitHub repositories
* README files
* URLs
* JSON
* CSV
* YAML
* API responses

Source ingestion may eventually become an adapter system.

The renderer should not care where data originated.

⸻

22. Storage

The current application stores projects and settings in browser localStorage.

This is appropriate for the current prototype.

The long-term source of truth should be the portable project specification.

Potential storage layers:

Studio
  └── local projects
Git
  └── project files
CI
  └── project files + generated artifacts

A server-side database is not required by the core architecture.

⸻

23. Repository Evolution

The current application is intentionally simple.

The project should evolve incrementally.

A possible future repository structure is:

infographic-studio/
│
├── apps/
│   └── studio/
│
├── packages/
│   ├── core/
│   ├── schema/
│   ├── primitives/
│   ├── patterns/
│   ├── templates/
│   ├── themes/
│   └── exporters/
│
├── cli/
│
├── action/
│
├── examples/
│
├── docs/
│
├── PRD.md
├── ARCHITECTURE.md
└── OPEN-SPEC.md

This is a target architecture, not an immediate refactoring requirement.

⸻

24. Migration Strategy

The migration should happen incrementally.

Phase 1 — Document

Create and stabilize:

* PRD
* architecture
* open specification

No major refactoring is required.

⸻

Phase 2 — Extract the model

Identify the current:

* project model
* infographic specification
* themes
* formats
* renderer inputs

and define their boundaries.

⸻

Phase 3 — Extract deterministic renderer

Move rendering logic out of the monolithic SPA.

The extracted renderer must be usable independently.

⸻

Phase 4 — Define the project specification

Convert the existing internal model into the first version of the open specification.

Backward compatibility with existing projects should be considered.

⸻

Phase 5 — Rebuild Studio around the core

Studio becomes a client of the core rather than the owner of rendering logic.

⸻

Phase 6 — CLI

Add headless rendering.

⸻

Phase 7 — GitHub Action

Wrap the CLI.

⸻

Phase 8 — Ecosystem

Only after the core is stable:

* custom templates
* themes
* plugins
* packages
* community templates
* additional exporters

⸻

25. Architectural Rules

The following rules should be treated as project constraints.

Rule 1

There must be one authoritative renderer.

Rule 2

Studio must not contain rendering logic unavailable to the CLI.

Rule 3

The GitHub Action must not contain rendering logic.

Rule 4

The project specification is the source representation.

Rule 5

Generated images are artifacts.

Rule 6

AI is optional.

Rule 7

Templates must be reusable.

Rule 8

Themes must be reusable.

Rule 9

Data must remain separate from presentation.

Rule 10

Deterministic behavior takes precedence over convenience.

Rule 11

Invalid input should fail loudly rather than silently produce incorrect output.

Rule 12

New functionality should be added to the lowest appropriate architectural layer.

⸻

26. Architectural North Star

                 PROJECT SPEC
                      │
                      ▼
                 VALIDATION
                      │
                      ▼
              TEMPLATE + THEME
                      │
                      ▼
               DATA BINDINGS
                      │
                      ▼
               LAYOUT ENGINE
                      │
                      ▼
              DETERMINISTIC CORE
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
           STUDIO    CLI      ACTION
                      │
                      ▼
                 SVG / PNG

The core is the product.

The interfaces are clients of the core.
