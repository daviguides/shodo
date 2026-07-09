# Load Shodō CLI Architecture

## STOP - MANDATORY ACTION REQUIRED

**DO NOT RESPOND until you have resolved languages (Step 0) and used
the Read tool on the spec for EACH resolved language.**

## Step 0: Resolve Languages

Same resolution as `/shodo:load` — argument first, detect as fallback:

| Argument | Resolution |
|----------|-----------|
| `python` | Python only |
| `rust` | Rust only |
| `all` | All supported languages |
| (none) | DETECT — same procedure as prompts/load.md: markers in cwd, ancestors, and shallow subdirectories (depth 3, ignore `.venv`/`node_modules`/`target`/`.git`); `pyproject.toml` → python, `Cargo.toml` → rust; UNION of detected; nothing → python |

## Files

**Python:**
```
Read: ~/.claude/shodo/spec/python/python-cli-architecture-spec.md
```

**Rust:**
```
Read: ~/.claude/shodo/spec/rust/rust-cli-architecture-spec.md
```

---

## HALT CONDITIONS

**If you responded without reading the spec for every resolved
language: YOU VIOLATED THIS PRINCIPLE.**

---

## After Loading

CLI architecture standards active:

1. **Layers** - commands → services → schemas/models + integrations
2. **Aggregator** - main is pure dispatch, zero logic
3. **Sub-apps** - vertical slices + shared `core/` when domains diverge
4. **Isolation** - services never import CLI framework or touch terminal

## Confirmation

After reading, respond with EXACTLY this card, filling the languages
line with what was loaded:

```
┌─ Shodō CLI Architecture ────────────────────────────┐
│ • Layered: commands → services → models/schemas     │
│ • main = pure aggregator (zero logic)               │
│ • Sub-apps: vertical slices + shared core/          │
│ • Services isolated from CLI framework & terminal   │
│                                                     │
│ Loaded: {languages}                                 │
└─────────────────────────────────────────────────────┘
```
