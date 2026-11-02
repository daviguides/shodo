# Shodō - Claude Code Project Instructions

## Project Overview

**Shodō** (書道) is the Way of Calligraphy - the art of beautiful, precise writing.

**Philosophy**: Like Japanese calligraphy masters who practice brush strokes for elegance and precision, Shodō guides developers to write code with clarity — language-specific standards for Python, Rust, Java, and TypeScript (style, types, errors, libraries, CLI architecture).

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
│   │   │   ├── python-cli-architecture-spec.md
│   │   │   └── python-make-spec.md
│   │   ├── rust/             # Rust standards
│   │   │   ├── rust-language-spec.md
│   │   │   ├── rust-style-spec.md
│   │   │   ├── rust-libraries-spec.md
│   │   │   ├── rust-testing-tools-spec.md
│   │   │   ├── rust-cli-architecture-spec.md
│   │   │   └── rust-make-spec.md
│   │   ├── java/             # Java standards
│   │   │   ├── java-language-spec.md
│   │   │   ├── java-style-spec.md
│   │   │   ├── java-libraries-spec.md
│   │   │   ├── java-testing-tools-spec.md
│   │   │   ├── java-cli-architecture-spec.md
│   │   │   └── java-make-spec.md
│   │   ├── make-contract-spec.md  # Component Makefile contract (all languages)
│   │   └── typescript/       # TypeScript standards (frontend)
│   │       ├── typescript-language-spec.md
│   │       ├── typescript-style-spec.md
│   │       ├── typescript-libraries-spec.md
│   │       ├── typescript-testing-tools-spec.md
│   │       ├── typescript-cli-architecture-spec.md
│   │       └── typescript-make-spec.md
│   ├── context/
│   │   └── examples/
│   │       ├── python-patterns.md
│   │       ├── python-templates.md
│   │       ├── python-anti-patterns.md
│   │       ├── rust-patterns.md
│   │       ├── rust-templates.md
│   │       ├── rust-anti-patterns.md
│   │       ├── java-patterns.md
│   │       ├── java-templates.md
│   │       ├── java-anti-patterns.md
│   │       ├── typescript-patterns.md
│   │       ├── typescript-templates.md
│   │       └── typescript-anti-patterns.md
│   └── prompts/
│       ├── load.md           # Language resolution + standards loading (interactive)
│       ├── load-cli.md       # CLI architecture loading (interactive)
│       ├── load-planning-<lang>.md  # Runner entry: PLANNING / plan review
│       ├── load-<lang>.md           # Runner entry: IMPLEMENTING
│       └── load-review-<lang>.md    # Runner entry: VALIDATING
├── commands/
├── skills/
└── docs/
```


## Commands

| Command | Purpose |
|---------|---------|
| `/shodo:load [python\|rust\|java\|typescript\|all]` | Load language standards (detects project languages if omitted) |
| `/shodo:load-cli [python\|rust\|java\|typescript\|all]` | Load CLI architecture standards (same resolution) |

**Language resolution**: explicit argument wins; without argument,
detect markers (`pyproject.toml` → python, `Cargo.toml` → rust,
`pom.xml`/`build.gradle`/`build.gradle.kts` → java,
`tsconfig.json` → typescript) in cwd,
ancestors, and shallow subdirectories, loading the UNION
(monorepos load multiple). Nothing detected → python fallback.

**TypeScript** is framed as a frontend language. Unlike python/rust/
java, its toolchain guidance is flexible: an existing project's
package manager, bundler, linter, formatter, and test runner are law;
new projects get the fastest option (native tooling). The "fastest per
category" table in typescript-style-spec is a dated snapshot —
re-verify it when touching the spec.


## Runner entries (per mode × language)

The runner compiles gradients into the system prompt, so the
interactive `load.md` (language detection in prose, every `Read:`
inlined) would ship all four languages. These thin entries list only
the files one mode needs for one language. No confirmation card: they
are compiled into a system prompt, not run as a command. `load.md`
and `load-cli.md` keep their interactive behavior.

`<lang>` ∈ python, rust, java, typescript.

| File | load-planning-<lang> | load-cli-<lang> | load-<lang> | load-review-<lang> | load-make-<lang> |
|------|:---:|:---:|:---:|:---:|:---:|
| spec/<lang>/<lang>-language-spec.md |  |  | ✓ |  |  |
| spec/<lang>/<lang>-style-spec.md |  |  | ✓ | ✓ |  |
| spec/<lang>/<lang>-libraries-spec.md | ✓ |  | ✓ |  |  |
| spec/<lang>/<lang>-testing-tools-spec.md |  |  | ✓ |  |  |
| spec/<lang>/<lang>-cli-architecture-spec.md |  | ✓ |  |  |  |
| context/examples/<lang>-patterns.md |  |  | ✓ |  |  |
| context/examples/<lang>-templates.md |  |  | ✓ |  |  |
| context/examples/<lang>-anti-patterns.md |  |  | ✓ | ✓ |  |
| spec/make-contract-spec.md |  |  |  |  | ✓ |
| spec/<lang>/<lang>-make-spec.md |  |  |  |  | ✓ |

Rationale: planning chooses libraries; implementing needs the full
idiom; review judges style and known anti-patterns. `load-cli-<lang>`
is conditional: the runner loads it only when faro detects the `cli`
aspect, in PLANNING, PLAN_REVIEWING and VALIDATING (not IMPLEMENTING:
the implementer follows the layers fixed in the plan). Not every task
touches a CLI, so the spec stays out of `load-planning-<lang>`.
`load-make-<lang>` carries the component Makefile contract (`test`
and `check` in the component's own `Makefile`, never the repo root):
the runner runs `make -C <component> test check` between
IMPLEMENTING and VALIDATING, and a task whose component lacks the
targets adds them. Loaded in PLANNING, PLAN_REVIEWING and
IMPLEMENTING.

Compiled size (chars, ~tokens at chars/3.6):

| Entry | chars | ~tokens |
|-------|------:|------:|
| `load-planning-python` | 8,947 | ~2.4k |
| `load-python` | 49,507 | ~13.7k |
| `load-review-python` | 11,581 | ~3.2k |
| `load-planning-rust` | 9,780 | ~2.7k |
| `load-rust` | 32,399 | ~8.9k |
| `load-review-rust` | 9,876 | ~2.7k |
| `load-planning-java` | 11,483 | ~3.1k |
| `load-java` | 38,849 | ~10.7k |
| `load-review-java` | 11,466 | ~3.1k |
| `load-planning-typescript` | 10,196 | ~2.8k |
| `load-typescript` | 39,044 | ~10.8k |
| `load-review-typescript` | 11,052 | ~3.0k |
| `load` (interactive, all 4 languages) | 164,463 | ~45.7k |

Measured with `compile_plugin("shodo", prompt=<entry>, strict=True)`
on v1.4.1.

Keep the entries in sync when a spec file is added to a language.


## Related Plugins

- **Zazen**: Core zen principles (naming, structure) — universal,
  always loaded alongside Shodō; language specs NEVER repeat zazen
- **Kinhin**: TDD practices — owns TypeScript TDD methodology; shodo's
  typescript-testing-tools-spec references it, never repeats it
- **Arche**: Behavioral principles for Claude Code
- **Gradient**: Plugin architecture


## Origin

Extracted from Zazen to separate concerns:
- **Zazen**: Zen principles + naming + structure
- **Shodō**: Language standards (style, types, errors, libraries) — Python, Rust, Java & TypeScript
- **Kinhin**: TDD practices

---

## Releasing — mandatory workflow

Every plugin modification MUST follow this sequence. No exceptions.

### 1. Bump version

Patch for fixes/tweaks, minor for new skills or behavioral changes:

```bash
# From gradients/shodo/
# Edit .claude-plugin/plugin.json version field
# Also update install.sh header if it shows a version
```

### 2. Commit and push

```bash
git add -A && git commit -m "bump: vX.Y.Z — <what changed>"
git tag -a vX.Y.Z -m "<what changed>"
git push && git push origin vX.Y.Z
```

### 3. Run install.sh

```bash
~/work/sources/continuum/gradients/shodo/install.sh
```

Note: install.sh clones from the GitHub remote (not local source), so
the push in step 2 must land before running it.

### 4. Verify cache is not stale

The plugin cache (`~/.claude/plugins/cache/daviguides/shodo/`) is
unstable — even after a successful install, it can preserve stale
state from previous versions. This is a known unresolved bug in the
Claude Code plugin system.

After install, always verify:

```bash
# Compare installed vs source timestamps
diff <(ls -lR ~/.claude/shodo/skills/) <(ls -lR skills/)

# Check cache version matches
ls ~/.claude/plugins/cache/daviguides/shodo/

# If stale, nuke cache and reinstall
rm -rf ~/.claude/plugins/cache/daviguides/shodo/
rm -rf ~/.claude/shodo/
./install.sh
```

Do NOT move to the next task with a stale cache — the session will
load outdated skills silently.
