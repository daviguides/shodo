# Java Library Preferences Specification

Java equivalents of the shodo Python stack — chosen for strong
typing, virtual-thread compatibility, and community consensus
(state of the ecosystem: 2026).

## Python → Java Mapping

| Python (shodo) | Java | Notes |
|----------------|------|-------|
| typer | **picocli** | Annotation-driven CLI, subcommands, ANSI |
| rich | **picocli ANSI + JLine 3** | No 1:1 — see below |
| fastapi | **Spring Boot 4** | Quarkus for K8s-native greenfield |
| pydantic | **records + Jakarta Validation + Jackson 3** | records = shape, validation = rules |
| pytest | **JUnit 6 + AssertJ** | See java-testing-tools-spec |
| httpx | **java.net.http.HttpClient** | Built-in, async, HTTP/3 (JDK 26) |
| logging | **SLF4J 2 + Logback** | Fluent API for structured events |
| litellm | **LangChain4j** | Official Anthropic/OpenAI SDKs for direct use |
| motor | **mongodb-driver-sync** | Blocking driver on virtual threads |

## CLI & Terminal

- **picocli** — CLI argument parsing, the absolute standard
  (annotations, nested subcommands, help/completion generation,
  GraalVM metadata via `picocli-codegen`)
- **The "rich stack"** — Java splits rich into focused pieces:
  - **picocli `Help.Ansi`** — colors and styled help (built-in)
  - **JLine 3** — line editing, prompts, terminals (REPLs only)
- Plain `System.out` + picocli ANSI covers most CLI output; do not
  add a TUI framework to print a table

## Web & APIs
- **Spring Boot 4** — enterprise APIs (Spring Framework 7,
  JSpecify null-safety, virtual threads, Jackson 3 default)
- **Quarkus** — container/K8s-native greenfield alternative
  (dev tooling, fast startup, native-image first-class)
- Virtual threads + blocking style over reactive stacks

## Data Validation & Modeling
- **records** — data shape (language-level, not a library)
- **Jakarta Bean Validation** (Hibernate Validator) — declarative
  rules at API boundaries
- **Jackson 3** (`tools.jackson.*`) — JSON serialization; Gson is
  maintenance-mode, do not adopt

## HTTP
- **java.net.http.HttpClient** — built-in, async, zero dependencies;
  OkHttp only when its interceptor ecosystem is genuinely needed

## Async & Concurrency
- **Virtual threads** (platform) — no reactive framework for I/O
  concurrency
- `CompletableFuture` at API seams; Scoped Values for context

## Database
- **HikariCP** — connection pool (universal)
- **jOOQ** — SQL-first, typed queries (reporting, complex SQL)
- **Spring Data JDBC** — simple aggregates in Spring services
- **Hibernate/JPA** — only for genuinely CRUD-heavy domain models
- **mongodb-driver-sync** — official MongoDB driver

## Logging & Observability
- **SLF4J 2** — the API, mandatory; fluent
  `log.atInfo().addKeyValue(...)` for structured events
- **Logback 1.5** — the backend (Boot default; native structured
  JSON in Spring Boot)

## LLM & AI
- **LangChain4j** — abstraction, RAG, tools, MCP
- **Official Anthropic / OpenAI Java SDKs** — direct API use
- **Spring AI** — Spring-native shops

## Explicitly Avoided

- **Lombok** — records + sealed types replaced it; internals-hacking
  breaks on new JDKs
- **Guava as default** — the JDK caught up (`List.of`, Streams,
  `Objects`); adopt only for a specific missing utility
- **Reactive frameworks for plain I/O** — virtual threads made
  blocking style the simple AND scalable option

## Selection Philosophy

### Priority Order
1. **Strong typing** — the API must leverage records/sealed types
2. **Virtual-thread compatibility** — no thread-pinning libraries
3. **JDK-first** — prefer the platform (HttpClient, `java.time`,
   Streams) over dependencies
4. **Active maintenance** — regular releases, current-LTS support
5. **Community adoption** — de-facto standards over novelty

### Avoid
- Unmaintained libraries (check last release before adopting)
- Annotation processors that hack compiler internals
- Heavy dependency trees for trivial needs (audit with
  `mvn dependency:tree`)
- Libraries without JSpecify/nullness annotations when alternatives
  have them
