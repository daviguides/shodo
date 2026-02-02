# Shodō

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> The Way of Python Calligraphy, designed as a Claude Code plugin.

## What is Shodō?

**Shodō** (書道) is the Japanese art of calligraphy - the Way of Writing. Like calligraphy masters who practice brush strokes for elegance and precision, Shodō guides developers to write Python with beauty and clarity.

```
       ●

 S  H  O  D  Ō

       │
```

Shodō provides **Python-specific standards** - style, type hints, docstrings, and library best practices.

## Installation

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/daviguides/shodo/main/install.sh)"
```

## Philosophy

### The Way of Beautiful Code (書道)

Just as calligraphy masters spend years perfecting each stroke, Shodō teaches the art of writing Python that is both functional and beautiful.

> *"The brush must move with both precision and freedom."*
> — Calligraphy wisdom

### Core Standards

| Standard | Calligraphy Virtue | Purpose |
|----------|-------------------|---------|
| **Style** | Form (形) | PEP 8, consistent formatting |
| **Type Hints** | Clarity (明) | Explicit types, self-documenting |
| **Docstrings** | Communication (伝) | Clear documentation |
| **Libraries** | Tools (具) | Proper use of Python ecosystem |

## Relationship with Other Plugins

Shodō is the calligraphy that gives form to Python code:

| Plugin | Philosophy | Focus |
|--------|------------|-------|
| **zazen** | Zen (座禅) | Universal principles (any language) |
| **shodo** | Calligraphy (書道) | Python standards |
| **kinhin** | Walking meditation (経行) | TDD practices |
| **arche** | Greek (ἀρχή) | LLM behavioral principles |

```
        ┌────────┐
        │ zazen  │  ← Universal principles
        └───┬────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
┌───────┐ ┌───────┐ ┌────────┐
│ shodo │ │kinhin │ │ kyudo  │
└───────┘ └───────┘ └────────┘
 Python     TDD      Actions
   ▲
   │
  YOU ARE HERE
```

## Project Structure

```
shodo/
├── spec/                    # Normative specifications
│   └── python/              # Python standards
│       ├── python-language-spec.md
│       ├── python-style-spec.md
│       ├── python-libraries-spec.md
│       └── python-testing-tools-spec.md
├── context/                 # Applied examples
│   └── examples/            # Python patterns
│       ├── python-patterns.md
│       ├── python-templates.md
│       └── python-anti-patterns.md
├── prompts/                 # Workflow orchestrators
├── commands/                # User-facing commands
└── skills/                  # Skill definitions
```

## Usage

```bash
/shodo:load                  # Load Python standards
```

## License

MIT License

---

> *"In calligraphy, each stroke carries meaning. In Python, each line carries intent."*
