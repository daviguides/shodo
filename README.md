# Shodō

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> The Way of Code Calligraphy, designed as a Claude Code plugin.

## What is Shodō?

**Shodō** (書道) is the Japanese art of calligraphy - the Way of Writing. Like calligraphy masters who practice brush strokes for elegance and precision, Shodō guides developers to write code with beauty and clarity.

```
       ●

 S  H  O  D  Ō

       │
```

Shodō provides **language-specific standards** for **Python, Rust, Java, and TypeScript** (frontend) - style, types, error handling, library preferences, and CLI architecture.

## Installation

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/daviguides/shodo/main/install.sh)"
```

## Philosophy

### The Way of Beautiful Code (書道)

Just as calligraphy masters spend years perfecting each stroke, Shodō teaches the art of writing code that is both functional and beautiful.

> *"The brush must move with both precision and freedom."*
> — Calligraphy wisdom

### Core Standards

| Standard | Calligraphy Virtue | Python | Rust | Java | TypeScript |
|----------|-------------------|--------|------|------|------------|
| **Style** | Form (形) | PEP 8 + Ruff | rustfmt + clippy | Spotless + Error Prone | Project's toolchain; new = fastest native |
| **Types** | Clarity (明) | Type hints | Newtypes, exhaustive match | records, sealed types | strict, discriminated unions, brands |
| **Errors** | Honesty (誠) | Explicit exceptions | Result + thiserror/anyhow | Typed exceptions, chained | Error + cause, Result unions at edges |
| **Docs** | Communication (伝) | Docstrings | Doc comments | Javadoc | TSDoc |
| **Libraries** | Tools (具) | typer, rich, pydantic | clap, serde, tokio | picocli, Jackson, JUnit | React, Valibot/ArkType, Vitest |

## Relationship with Other Plugins

Shodō is the calligraphy that gives form to code:

| Plugin | Philosophy | Focus |
|--------|------------|-------|
| **zazen** | Zen (座禅) | Universal principles (any language) |
| **shodo** | Calligraphy (書道) | Language standards (Python, Rust, Java & TypeScript) |
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
 Python,    TDD      Actions
Rust, Java,
TypeScript
   ▲
   │
  YOU ARE HERE
```

## Project Structure

```
shodo/
├── spec/                    # Normative specifications
│   ├── python/              # Python standards
│   │   ├── python-language-spec.md
│   │   ├── python-style-spec.md
│   │   ├── python-libraries-spec.md
│   │   ├── python-testing-tools-spec.md
│   │   └── python-cli-architecture-spec.md
│   ├── rust/                # Rust standards
│   │   ├── rust-language-spec.md
│   │   ├── rust-style-spec.md
│   │   ├── rust-libraries-spec.md
│   │   ├── rust-testing-tools-spec.md
│   │   └── rust-cli-architecture-spec.md
│   ├── java/                # Java standards
│   │   ├── java-language-spec.md
│   │   ├── java-style-spec.md
│   │   ├── java-libraries-spec.md
│   │   ├── java-testing-tools-spec.md
│   │   └── java-cli-architecture-spec.md
│   └── typescript/          # TypeScript standards (frontend)
│       ├── typescript-language-spec.md
│       ├── typescript-style-spec.md
│       ├── typescript-libraries-spec.md
│       ├── typescript-testing-tools-spec.md
│       └── typescript-cli-architecture-spec.md
├── context/                 # Applied examples
│   └── examples/
│       ├── python-patterns.md / templates / anti-patterns
│       ├── rust-patterns.md / templates / anti-patterns
│       ├── java-patterns.md / templates / anti-patterns
│       └── typescript-patterns.md / templates / anti-patterns
├── prompts/                 # Workflow orchestrators
├── commands/                # User-facing commands
└── skills/                  # Skill definitions
```

## Usage

```bash
/shodo:load                  # Detect project languages and load standards
/shodo:load rust             # Load Rust standards only
/shodo:load python           # Load Python standards only
/shodo:load java             # Load Java standards only
/shodo:load typescript       # Load TypeScript standards only
/shodo:load all              # Load every supported language

/shodo:load-cli              # Load CLI architecture (same resolution)
```

Without an argument, Shodō detects languages by project markers
(`pyproject.toml` → Python, `Cargo.toml` → Rust, `pom.xml` /
`build.gradle` / `build.gradle.kts` → Java, `tsconfig.json` →
TypeScript) — in the current
directory, ancestors, and shallow subdirectories — and loads the
union (monorepos load multiple languages).

TypeScript is framed as a frontend language, and its toolchain
guidance is flexible: an existing project's package manager, bundler,
linter, formatter, and test runner are always respected; new projects
get the fastest option, which in practice is native tooling (Rust/Go/
Zig) — the same reason the Python stack is uv, ruff, and pydantic.

## License

MIT License

---

> *"In calligraphy, each stroke carries meaning. In code, each line carries intent."*
