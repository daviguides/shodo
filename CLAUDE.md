# Shodō - Claude Code Project Instructions

## Project Overview

**Shodō** (書道) is the Way of Calligraphy - the art of beautiful, precise writing.

**Philosophy**: Like Japanese calligraphy masters who practice brush strokes for elegance and precision, Shodō guides developers to write code with clarity — language-specific standards for Python and Rust (style, types, errors, libraries, CLI architecture).

```
       ●

 S  H  O  D  Ō

       │
```


## Structure

```
shodo/
├── shodo/                    # Bundle (Gradient pattern)
│   ├── spec/
│   │   ├── python/           # Python standards
│   │   │   ├── python-language-spec.md
│   │   │   ├── python-style-spec.md
│   │   │   ├── python-libraries-spec.md
│   │   │   ├── python-testing-tools-spec.md
│   │   │   └── python-cli-architecture-spec.md
│   │   └── rust/             # Rust standards
│   │       ├── rust-language-spec.md
│   │       ├── rust-style-spec.md
│   │       ├── rust-libraries-spec.md
│   │       ├── rust-testing-tools-spec.md
│   │       └── rust-cli-architecture-spec.md
│   ├── context/
│   │   └── examples/
│   │       ├── python-patterns.md
│   │       ├── python-templates.md
│   │       ├── python-anti-patterns.md
│   │       ├── rust-patterns.md
│   │       ├── rust-templates.md
│   │       └── rust-anti-patterns.md
│   └── prompts/
│       ├── load.md           # Language resolution + standards loading
│       └── load-cli.md       # CLI architecture loading
├── commands/
├── skills/
└── docs/
```


## Commands

| Command | Purpose |
|---------|---------|
| `/shodo:load [python\|rust\|all]` | Load language standards (detects project languages if omitted) |
| `/shodo:load-cli [python\|rust\|all]` | Load CLI architecture standards (same resolution) |

**Language resolution**: explicit argument wins; without argument,
detect markers (`pyproject.toml` → python, `Cargo.toml` → rust) in
cwd, ancestors, and shallow subdirectories, loading the UNION
(monorepos load multiple). Nothing detected → python fallback.


## Related Plugins

- **Zazen**: Core zen principles (naming, structure) — universal,
  always loaded alongside Shodō; language specs NEVER repeat zazen
- **Kinhin**: TDD practices
- **Arche**: Behavioral principles for Claude Code
- **Gradient**: Plugin architecture


## Origin

Extracted from Zazen to separate concerns:
- **Zazen**: Zen principles + naming + structure
- **Shodō**: Language standards (style, types, errors, libraries) — Python & Rust
- **Kinhin**: TDD practices
