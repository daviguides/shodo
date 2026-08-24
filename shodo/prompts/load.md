# Load Shodō Standards

## STOP - MANDATORY ACTION REQUIRED

**DO NOT RESPOND until you have resolved languages (Step 0) and used
the Read tool on EVERY file for EACH resolved language.**

## Step 0: Resolve Languages

An argument may have been passed (e.g. `/shodo:load rust`):

| Argument | Resolution |
|----------|-----------|
| `python` | Python only |
| `rust` | Rust only |
| `java` | Java only |
| `all` | All supported languages |
| (none) | DETECT — procedure below |

### DETECT Procedure

1. Search for markers in the current directory, its ancestors, AND
   shallow subdirectories (max depth 3). Use Glob or:
   ```bash
   find . -maxdepth 3 \( -name pyproject.toml -o -name Cargo.toml \
     -o -name pom.xml -o -name build.gradle -o -name build.gradle.kts \
     -o -name tsconfig.json \) \
     -not -path "*/node_modules/*" -not -path "*/.venv/*" \
     -not -path "*/target/*" -not -path "*/.git/*" 2>/dev/null
   ```
2. Map markers to languages:

   | Marker | Language |
   |--------|----------|
   | `pyproject.toml` | python |
   | `Cargo.toml` | rust |
   | `pom.xml` / `build.gradle` / `build.gradle.kts` | java |
   | `tsconfig.json` | typescript (no specs yet — mention and skip) |

3. Load the **UNION** of all detected languages (monorepos load
   multiple languages).
4. **Nothing detected → fallback: python.**

## Python Files (7)

### Step 1: Read Python Specs
```
Read: ~/.claude/shodo/spec/python/python-language-spec.md
Read: ~/.claude/shodo/spec/python/python-style-spec.md
Read: ~/.claude/shodo/spec/python/python-libraries-spec.md
Read: ~/.claude/shodo/spec/python/python-testing-tools-spec.md
```

### Step 2: Read Python Examples
```
Read: ~/.claude/shodo/context/examples/python-patterns.md
Read: ~/.claude/shodo/context/examples/python-templates.md
Read: ~/.claude/shodo/context/examples/python-anti-patterns.md
```

## Rust Files (7)

### Step 1: Read Rust Specs
```
Read: ~/.claude/shodo/spec/rust/rust-language-spec.md
Read: ~/.claude/shodo/spec/rust/rust-style-spec.md
Read: ~/.claude/shodo/spec/rust/rust-libraries-spec.md
Read: ~/.claude/shodo/spec/rust/rust-testing-tools-spec.md
```

### Step 2: Read Rust Examples
```
Read: ~/.claude/shodo/context/examples/rust-patterns.md
Read: ~/.claude/shodo/context/examples/rust-templates.md
Read: ~/.claude/shodo/context/examples/rust-anti-patterns.md
```

## Java Files (7)

### Step 1: Read Java Specs
```
Read: ~/.claude/shodo/spec/java/java-language-spec.md
Read: ~/.claude/shodo/spec/java/java-style-spec.md
Read: ~/.claude/shodo/spec/java/java-libraries-spec.md
Read: ~/.claude/shodo/spec/java/java-testing-tools-spec.md
```

### Step 2: Read Java Examples
```
Read: ~/.claude/shodo/context/examples/java-patterns.md
Read: ~/.claude/shodo/context/examples/java-templates.md
Read: ~/.claude/shodo/context/examples/java-anti-patterns.md
```

---

## HALT CONDITIONS

**If you responded without reading all files for every resolved
language: YOU VIOLATED THIS PRINCIPLE.**

---

## After Loading

**Python** standards active:
1. **Type Hints** - All functions must have type annotations
2. **Docstrings** - Google-style docstrings required
3. **Style** - Ruff formatting, 80 char lines
4. **Libraries** - Modern Python 3.13+ patterns
5. **Testing** - pytest + hypothesis standards

**Rust** standards active:
1. **Ownership** - Borrow by default, no unwrap on recoverable paths
2. **Errors** - thiserror (libs) + anyhow (bins), propagate with `?`
3. **Style** - rustfmt + clippy -D warnings
4. **Language** - Edition 2024, stable toolchain, newtypes
5. **Testing** - cargo-nextest + rstest + proptest

**Java** standards active:
1. **Data** - records + sealed interfaces + pattern matching, no Lombok
2. **Nullness** - JSpecify @NullMarked + NullAway, Optional at returns
3. **Style** - Spotless (palantir-java-format) + Error Prone
4. **Language** - Java 25 LTS, virtual threads for I/O
5. **Testing** - JUnit 6 + AssertJ + Testcontainers + ArchUnit

## Confirmation

After reading all files, respond with the card below. Include ONE
standards box PER loaded language, and fill the Files line with the
actual languages and counts (e.g. `python (7) + rust (7) = 14 files`):

```
╭─────────────────────────────────────────────────────╮
│                                                     │
│        ●                                            │
│                 │  The Way of Code Calligraphy      │
│  S  H  O  D  Ō  │  書道 - Elegant Code Standards    │
│                 │  (v1.2.0)                         │
│        │                                            │
│                                                     │
╰─────────────────────────────────────────────────────╯

┌─ Python Standards ──────────────────────────────────┐
│ • Type hints (all functions annotated)              │
│ • Google-style docstrings                           │
│ • Ruff formatting (80 char lines)                   │
│ • Python 3.13+ patterns                             │
│ • pytest + hypothesis testing                       │
└─────────────────────────────────────────────────────┘

┌─ Rust Standards ────────────────────────────────────┐
│ • Ownership: borrow by default, no unwrap           │
│ • thiserror (libs) + anyhow (bins) errors           │
│ • rustfmt + clippy -D warnings                      │
│ • Edition 2024, stable toolchain, newtypes          │
│ • cargo-nextest + rstest + proptest                 │
└─────────────────────────────────────────────────────┘

┌─ Java Standards ────────────────────────────────────┐
│ • records + sealed types + pattern matching         │
│ • JSpecify @NullMarked + NullAway, no Lombok        │
│ • Spotless (palantir) + Error Prone                 │
│ • Java 25 LTS, virtual threads for I/O              │
│ • JUnit 6 + AssertJ + Testcontainers + ArchUnit     │
└─────────────────────────────────────────────────────┘

┌─ Files ─────────────────────────────────────────────┐
│ {language (N) + language (N)} = {total} files loaded│
└─────────────────────────────────────────────────────┘
```

Omit the box of any language NOT loaded.
