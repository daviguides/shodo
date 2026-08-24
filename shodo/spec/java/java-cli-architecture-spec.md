# Java CLI Architecture Specification

Java materialization of the shodo CLI architecture. Same layers and
dependency rules as the Python spec — enforced by ArchUnit tests
where Python relies on discipline.

## Package Structure

```
project-name/
├── pom.xml
├── mvnw / .mvn/
├── src/main/java/com/example/toolname/
│   ├── Main.java            # Entrypoint — picocli bootstrap only
│   ├── cli/                 # picocli command definitions
│   ├── commands/            # Interface layer (command runners)
│   ├── services/            # Business logic layer
│   ├── model/               # records (data representation)
│   ├── integrations/        # Low-level I/O (fs, subprocess, APIs)
│   └── display/             # Semantic terminal output
├── src/main/resources/
│   └── data/                # Bundled config (classpath resources)
└── src/test/java/
```

## Layered Architecture

Five layers with strict dependency direction:

```
cli/ → commands/ → services/ → model/
                             → integrations/
                             → resources (data/)
```

### Layer Definitions

**`Main.java`** — Pure entrypoint. Builds `CommandLine`, sets the
exit-code mapper, executes. Zero logic. Under 40 lines.

```java
public final class Main {
    public static void main(String[] args) {
        int exitCode = new CommandLine(new RootCommand()).execute(args);
        System.exit(exitCode);
    }
}
```

**`cli/`** — picocli definitions only. `@Command` classes with
`@Option`/`@Parameters` fields and help text. Each `call()` is one
line: translate parsed args into a `commands/` runner call. No
behavior beyond dispatch.

**`commands/`** — Interface layer. Translates typed args into
service calls, formats output via `display/`, maps domain
exceptions to exit codes. Never executes domain logic directly.

**`services/`** — Business logic. Pure classes operating on domain
concepts. **No picocli imports. No `System.out`.** Receives records,
returns records or throws typed exceptions.

**`model/`** — records defining data shape. Pure representation —
validation in compact constructors, no I/O, no framework imports.

**`integrations/`** — Low-level operations. Filesystem
(`java.nio.file`), subprocess (`ProcessBuilder` wrappers for git/gh),
HTTP (`java.net.http`), YAML/JSON I/O. Domain-agnostic.

**`data/` (resources)** — Bundled config read via
`getResourceAsStream("/data/...")`. Not user-editable at runtime.

**`display/`** — Semantic output (`printSuccess`, `printError`,
`printWarning`) over picocli `Help.Ansi`. Imported by `commands/`
only. Errors to stderr, results to stdout.

## Dependency Rules

| Layer | Can import from | Cannot import from |
|-------|----------------|-------------------|
| `cli/` | `commands/`, picocli | `services/`, `integrations/` |
| `commands/` | `services/`, `model/`, `display/` | picocli (beyond exit codes) |
| `services/` | `model/`, `integrations/` | `cli/`, `commands/`, `display/`, **picocli** |
| `model/` | JDK, jspecify | anything else |
| `integrations/` | JDK, third-party | `cli/`, `commands/`, `services/`, `model/` |
| `display/` | picocli ANSI | `commands/`, `services/` |

Enforce with ArchUnit — the rules are tests, not review comments
(see java-testing-tools-spec).

## Entrypoint Convention

```java
// cli/RootCommand.java — picocli definitions only
@Command(
    name = "rover",
    mixinStandardHelpOptions = true,
    version = "rover 0.1.0",
    description = "Git repo scanner: pull, status, sync.",
    subcommands = {ScanCommand.class, SyncCommand.class})
public class RootCommand {}

@Command(name = "scan", description = "Scan repos for uncommitted changes.")
class ScanCommand implements Callable<Integer> {

    @Option(names = "--all", description = "Include repos with no remote.")
    boolean all;

    @Override
    public Integer call() {
        return new ScanRunner().run(new ScanRequest(all));
    }
}
```

## Sub-App Architecture (Context Separation)

The flat layout is the starting point. When command groups develop
distinct domains — services clustering by theme, records used by only
one group — graduate to **sub-apps**: vertical slices as subpackages
that replicate the layers internally.

```
com/example/toolname/
├── Main.java
├── display/             # Shared across all sub-apps
│
├── core/                # Sub-app for SHARED domain
│   ├── model/
│   ├── integrations/
│   └── services/
│
├── commit/              # Sub-app = vertical slice
│   ├── cli/             # Its own @Command classes
│   ├── commands/
│   ├── services/
│   └── model/
├── pull/                # Smaller slice — only the layers it needs
│   ├── cli/
│   ├── commands/
│   └── services/
└── sync/
```

### Sub-App Rules

1. **Each sub-app replicates only the layers it needs** — a slice
   with two layers is correct, not incomplete. Dependency rules apply
   WITHIN the slice.
2. **`core/` owns the shared domain** — sub-apps import from `core/`,
   **never from each other**. Logic needed by two slices moves to core.
3. **Top-level shared**: `Main.java`, `display/` — transversal to all
   slices.
4. **Subcommands compose on the root command**:

```java
@Command(subcommands = {
    commit.cli.CommitCommand.class,
    pull.cli.PullCommand.class,
    sync.cli.SyncCommand.class,
})
public class RootCommand {}
```

### When to Graduate

| Signal | Action |
|--------|--------|
| Command groups share no services | Split into sub-apps |
| `services/` clusters by theme | Each cluster becomes a slice |
| A record is used by one group only | It belongs inside that slice |
| Two slices need the same service | Move it to `core/` |

## Distribution

- **GraalVM native-image** is the target for CLI distribution —
  instant startup, no JRE; `picocli-codegen` annotation processor
  generates the reachability metadata
- **jlink/jpackage** fallback when reflection-heavy dependencies
  block native-image
- Never require users to run `java -jar`

## pom.xml Convention

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>project-name</artifactId>
    <version>0.1.0</version>

    <properties>
        <maven.compiler.release>25</maven.compiler.release>
    </properties>

    <dependencies>
        <dependency>
            <groupId>info.picocli</groupId>
            <artifactId>picocli</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jspecify</groupId>
            <artifactId>jspecify</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- spotless (palantir), error-prone + nullaway,
                 surefire (junit 6), jacoco, native-maven-plugin -->
        </plugins>
    </build>
</project>
```

## API Analogy

| API Concept (Spring) | CLI Equivalent |
|----------------------|---------------|
| `@RestController` routing | `cli/` (@Command classes) |
| handler methods | `commands/` |
| `@Service` | `services/` |
| DTO records | `model/` |
| clients/repositories | `integrations/` |
| `src/main/resources/static` | `resources/data/` |
| logging/filters | `display/` |
