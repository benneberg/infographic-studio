Infographic Studio — Open Specification

Status: Draft / Experimental
Specification version: 0.1

⸻

1. Purpose

The Infographic Studio Open Specification defines a portable, deterministic representation of an infographic project.

The specification is intended to allow different tools to:

* create infographic projects
* validate projects
* render projects
* preview projects
* store projects in Git
* process projects in CI/CD
* exchange projects between implementations

The specification is implementation-independent.

A compatible renderer should not require the Infographic Studio UI.

⸻

2. Design Principles

2.1 Declarative

A project describes what should be rendered, rather than providing imperative rendering instructions.

⸻

2.2 Deterministic

A project must contain enough information for a compatible renderer to reproduce the intended output.

⸻

2.3 Versioned

The specification, templates, themes, and renderer are independently versioned.

⸻

2.4 Human-readable

The preferred source representation should be readable and editable by humans.

YAML and JSON are both valid serialization formats.

⸻

2.5 Extensible

New template types, patterns, configuration properties, and output formats may be introduced without invalidating existing projects.

⸻

2.6 Explicit

Important behavior should be represented explicitly rather than inferred from arbitrary strings.

⸻

3. Conceptual Model

An infographic project consists of:

Project
├── metadata
├── canvas
├── template
├── theme
├── data
├── content
├── options
└── output

Conceptually:

Project
   │
   ├── Template ─────── structure
   │
   ├── Theme ────────── visual system
   │
   ├── Data ─────────── facts
   │
   ├── Content ──────── human-facing text
   │
   └── Options ──────── controlled variation

⸻

4. Minimal Project

A minimal project may look like:

specVersion: "0.1"
project:
  name: weekly-report
canvas:
  width: 1200
  height: 630
template:
  id: metrics-summary
  version: "1"
theme:
  id: technical
  version: "1"
data:
  metrics:
    - label: Downloads
      value: 1240
    - label: Users
      value: 847
content:
  title: Weekly Project Report

This is an illustrative example and is not yet a normative schema.

⸻

5. Specification Version

Every project must declare its specification version.

Example:

specVersion: "0.1"

The specification version describes the structure and semantics of the project file.

It is independent of:

* template version
* theme version
* renderer version

⸻

6. Project Metadata

Example:

project:
  id: weekly-report
  name: Weekly Project Report
  description: Engineering metrics for the current week

Potential metadata fields include:

id
name
description
author
license
created
modified
tags

Metadata must not affect visual rendering unless explicitly referenced by the template.

⸻

7. Canvas

The canvas defines the output dimensions.

Example:

canvas:
  width: 1200
  height: 630

Optional future properties may include:

canvas:
  width: 1200
  height: 630
  background: transparent

Aspect presets may be represented by templates or Studio conveniences rather than becoming mandatory parts of the core specification.

⸻

8. Template Reference

A project references a template.

template:
  id: metrics-summary
  version: "1"

A template defines:

* structure
* supported data
* patterns
* bindings
* layout constraints
* configuration options

The project should not duplicate the complete template definition unless explicitly creating an embedded/self-contained project.

⸻

9. Theme Reference

A project references a theme.

theme:
  id: technical
  version: "1"

Themes define visual properties such as:

colors
typography
spacing
radii
borders
shadows
chart styles

⸻

10. Data

Data contains factual information.

Example:

data:
  metrics:
    - id: downloads
      label: Downloads
      value: 1240
    - id: users
      label: Users
      value: 847

The data model should support structured values.

Potential types include:

string
number
boolean
null
object
array

The renderer must not silently change factual values.

⸻

11. Content

Content contains human-facing copy.

Example:

content:
  title: Weekly Project Report
  subtitle: Week 35
  footer: Updated automatically

Content may contain references to data where explicitly supported.

Example:

content:
  title: "{{data.projectName}}"

The exact binding syntax is not yet finalized.

⸻

12. Bindings

Bindings connect template slots to project data.

Conceptually:

template.slot
      ↓
data.metrics[0].value

A binding must be explicit.

The specification should avoid implicit “guess what the user meant” behavior.

⸻

13. Configuration

Templates may expose bounded configuration.

Example:

options:
  showTrend: true
  showFooter: true
  columns: 3

Configuration must be defined by the referenced template.

Arbitrary unknown options should produce a validation warning or error according to the compatibility rules.

⸻

14. Controlled Variation

Templates may support controlled variation.

For example:

options:
  variant: compact

If deterministic pseudo-random variation is eventually supported:

options:
  seed: 12345

The seed becomes part of the rendering input.

A renderer must never use an implicit random seed.

⸻

15. Assets

Projects may reference assets such as:

* logos
* icons
* illustrations
* images
* fonts

Example:

assets:
  logo:
    path: ./assets/logo.svg

Assets should preferably be local or otherwise versioned.

A future content-addressed asset mechanism may be introduced.

⸻

16. Output

Output configuration may eventually be represented in the project.

Example:

output:
  formats:
    - svg
    - png

However, output format may also remain a CLI/runtime concern.

The source project should remain independent of a particular output format wherever possible.

⸻

17. Templates

A template is a reusable visual composition.

Conceptually:

Template
├── metadata
├── supported configuration
├── slots
├── patterns
├── bindings
└── constraints

A template should define semantic requirements rather than expose every rendering primitive directly.

⸻

18. Patterns

Patterns are reusable semantic components.

Examples:

StatCard
MetricRow
TimelineStep
ComparisonCard
Callout
Badge
ProgressBar
RankingItem

A pattern may itself contain primitives and layout rules.

Patterns should be reusable between templates.

⸻

19. Primitives

Primitives are low-level visual elements.

Initial conceptual set:

Text
Rect
Circle
Line
Path
Image
Icon
Number
Chart

The exact primitive vocabulary is implementation-defined until the renderer has been extracted from the current application.

⸻

20. Layout

The specification should describe layout semantically wherever possible.

Examples:

stack
row
column
grid
overlay
timeline
flow

Templates may expose bounded layout configuration.

Arbitrary absolute positioning should not be required for normal template authoring.

⸻

21. Constraints

Templates may define constraints.

Examples:

constraints:
  metrics:
    minItems: 1
    maxItems: 6
  title:
    maxLines: 2

Constraints are important because deterministic rendering cannot rely on a human designer fixing every unexpected input.

⸻

22. Overflow

Overflow behavior must be explicit.

Possible strategies include:

wrap
truncate
shrink
expand
error

A template should define acceptable behavior.

For example:

title:
  overflow: shrink
  minFontSize: 18

The implementation must not silently choose an incompatible strategy.

⸻

23. Validation

A compatible implementation should be capable of validating a project before rendering.

Validation categories include:

Schema
Data
Template
Theme
Assets
Bindings
Constraints
Compatibility

Example:

ERROR E1001
Required field "content.title" is missing.
ERROR E2003
Template "metrics-summary" requires at least one metric.
WARNING W3002
Title exceeded preferred length and will be reduced to 18px.

⸻

24. Deterministic Rendering Contract

For a fixed:

project
template version
theme version
renderer version
assets
fonts

the renderer should produce the same output.

Formally:

R(P, T, H, A, F, V) = O

Where:

P = project
T = template
H = theme
A = assets
F = fonts
V = renderer version
O = output

If any rendering input changes, the output may change.

⸻

25. Renderer Version

The renderer version should be externally identifiable.

Example generated metadata:

generator:
  name: infographic-studio
  version: "0.5.0"

This allows generated artifacts to be traced back to a renderer implementation.

⸻

26. Accessibility

A compatible renderer should support accessible metadata where applicable.

For SVG:

<title>Weekly Project Report</title>
<desc>
  Project downloads increased to 1240 this week.
</desc>

Projects may eventually contain explicit accessibility content:

accessibility:
  title: Weekly Project Report
  description: Summary of project activity during week 35.
  altText: Weekly project report showing 1240 downloads and 847 users.

⸻

27. Security

Templates should preferably be declarative.

The specification should not require arbitrary executable JavaScript.

This is particularly important for:

* GitHub Actions
* CI environments
* third-party templates
* community template repositories

Executable templates may eventually exist as an advanced extension, but they should not be part of the initial core specification.

⸻

28. Compatibility

Implementations should declare supported specification versions.

Example:

Supports:
  0.1
  0.2

A renderer encountering a newer unsupported specification version should fail clearly rather than guessing.

⸻

29. Extensions

Future specifications may support extension namespaces.

Conceptually:

extensions:
  com.example.analytics:
    ...

Extensions must not redefine core semantics.

The extension mechanism should be designed only when real use cases emerge.

⸻

30. JSON Representation

YAML is used in examples because it is convenient for humans.

JSON is equally valid.

Example:

{
  "specVersion": "0.1",
  "project": {
    "name": "weekly-report"
  },
  "canvas": {
    "width": 1200,
    "height": 630
  },
  "template": {
    "id": "metrics-summary",
    "version": "1"
  },
  "theme": {
    "id": "technical",
    "version": "1"
  }
}

⸻

31. File Naming

Recommended project files:

infographic.yml
infographic.yaml
infographic.json

The file name is a convention and is not semantically significant.

⸻

32. Self-Contained Projects

Future versions may support a fully self-contained project:

project/
├── infographic.yml
├── assets/
│   ├── logo.svg
│   └── icon.svg
└── data/
    └── metrics.json

This makes projects especially suitable for Git repositories and CI/CD.

⸻

33. Template Packages

A future template package could contain:

template/
├── template.yml
├── preview.svg
├── README.md
└── assets/

Templates should be versioned independently.

⸻

34. Theme Packages

A future theme package could contain:

theme/
├── theme.yml
├── preview.svg
└── assets/

Themes should be reusable across templates.

⸻

35. Specification Evolution

The first specification must remain intentionally small.

The project should not attempt to define a complete universal graphics language in version 0.1.

New concepts should be introduced only after they are demonstrated by real templates and renderer requirements.

The implementation should prefer:

real template
    ↓
repeated requirement
    ↓
general abstraction
    ↓
specification feature

rather than:

theoretical abstraction
    ↓
large DSL
    ↓
implementation complexity

⸻

36. Current Status

Version 0.1 is an architectural draft.

The following are intentionally not yet frozen:

* exact JSON Schema
* binding syntax
* primitive vocabulary
* layout DSL
* template package format
* theme package format
* asset specification
* extension mechanism
* renderer metadata format
* visual regression format

These should be derived from the existing renderer and a small set of representative templates before being declared normative.

⸻

37. Specification North Star

The specification should make this possible:

                 infographic.yml
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Studio         CLI       Other Tool
          │            │            │
          └────────────┼────────────┘
                       ▼
                 Compatible
                  Renderer
                       │
                       ▼
               Deterministic SVG

The ultimate goal is not to make one application that happens to generate infographics.

The goal is to define a portable, deterministic representation of visual information that multiple tools can understand and render consistently.
