# Rust Library Preferences Specification

Rust equivalents of the shodo Python stack — chosen for performance,
async support, and strong typing.

## Python → Rust Mapping

| Python (shodo) | Rust | Notes |
|----------------|------|-------|
| typer | **clap** (derive) | Declarative CLI via derive macros |
| rich | **owo-colors + indicatif + comfy-table** | No 1:1 — see below |
| fastapi | **axum** | Async, tokio-native, type-safe extractors |
| pydantic | **serde** (+ **garde** for validation) | serde = shape, garde = rules |
| polars | **polars** | Same library — polars IS Rust |
| pytest | **rstest + proptest** | See rust-testing-tools-spec |
| httpx | **reqwest** | Async HTTP client |
| logging | **tracing** | Structured, async-aware |
| motor | **mongodb** (official driver) | tokio-based async |

## CLI & Terminal

- **clap** (derive API) — CLI argument parsing, the absolute standard
- **The "rich stack"** — Rust splits rich into focused crates;
  there is deliberately no 1:1 equivalent:
  - **owo-colors** — colors and styles (zero-allocation)
  - **indicatif** — progress bars and spinners
  - **comfy-table** — tables
- **ratatui** — full TUI apps only (a different level than
  "pretty output"; do not reach for it to print a table)

## Web & APIs
- **axum** — async APIs (preferred over actix-web/rocket)
  - **Rationale**: tokio-native, tower middleware, type-safe extractors

## Data Validation & Modeling
- **serde** — serialization/deserialization (universal)
- **garde** — validation rules on top of serde structs

## Errors
- **thiserror** — typed error enums (libraries)
- **anyhow** — contextual error chains (binaries)

## Async Runtime
- **tokio** — the async runtime (preferred over async-std)
- Use `#[tokio::main]` in binaries; keep library crates
  runtime-agnostic when feasible

## Data Processing
- **polars** — DataFrames (lazy evaluation, same engine the Python
  stack already standardizes on)

## HTTP
- **reqwest** — async HTTP client (rustls over native-tls when possible)

## Logging & Observability
- **tracing** + **tracing-subscriber** — structured, span-based
  (preferred over log/env_logger)

## Database
- **sqlx** — SQL, async, compile-time checked queries
- **mongodb** — official async driver

## Testing
- **rstest**, **proptest**, **criterion**, **insta**, **mockall**
- Details: rust-testing-tools-spec.md

## Selection Philosophy

### Priority Order
1. **Strong typing** — the API must leverage the type system
2. **Async support** — tokio-compatible when doing I/O
3. **Performance** — zero-cost, minimal allocations
4. **Active maintenance** — regular releases
5. **Community adoption** — de-facto standards over novelty

### Avoid
- Unmaintained crates (check last release before adopting)
- Crates that force a different async runtime
- Heavy dependency trees for trivial needs (audit with `cargo tree`)
- `unsafe`-heavy crates without strong justification and audit history
