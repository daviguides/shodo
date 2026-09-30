# Shodō - Claude Code Project Instructions

## Project Overview

**Shodō** (書道) is the Way of Calligraphy - the art of beautiful, precise writing.

**Philosophy**: Like Japanese calligraphy masters who practice brush strokes for elegance and precision, Shodō guides developers to write code with clarity — language-specific standards for Python, Rust, and Java (style, types, errors, libraries, CLI architecture).

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
│   │   ├── rust/             # Rust standards
│   │   │   ├── rust-language-spec.md
│   │   │   ├── rust-style-spec.md
│   │   │   ├── rust-libraries-spec.md
│   │   │   ├── rust-testing-tools-spec.md
│   │   │   └── rust-cli-architecture-spec.md
│   │   └── java/             # Java standards
│   │       ├── java-language-spec.md
│   │       ├── java-style-spec.md
│   │       ├── java-libraries-spec.md
│   │       ├── java-testing-tools-spec.md
│   │       └── java-cli-architecture-spec.md
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
│   │       └── java-anti-patterns.md
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
| `/shodo:load [python\|rust\|java\|all]` | Load language standards (detects project languages if omitted) |
| `/shodo:load-cli [python\|rust\|java\|all]` | Load CLI architecture standards (same resolution) |

**Language resolution**: explicit argument wins; without argument,
detect markers (`pyproject.toml` → python, `Cargo.toml` → rust,
`pom.xml`/`build.gradle`/`build.gradle.kts` → java) in cwd,
ancestors, and shallow subdirectories, loading the UNION
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
- **Shodō**: Language standards (style, types, errors, libraries) — Python, Rust & Java
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
