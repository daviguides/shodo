# Java Code Templates

## Service Method Template

```java
/**
 * {Brief description of what this method does}.
 *
 * @throws {DomainException} when {condition}
 */
public {ReturnType} {methodName}({Type1} {param1}, {Type2} {param2}) {
    // Guard clauses first (flat > nested)
    if (!{precondition}) {
        throw new {DomainException}("{context}: " + {param1});
    }

    var {value} = {fallibleLookup}
        .orElseThrow(() -> new {NotFoundException}({param1}));

    // Main logic
    var {intermediate} = {stepOne}({value});
    return {stepTwo}({intermediate});
}
```

## Record Template (Validated Value Type)

```java
/** {What this value represents}. */
public record {TypeName}({Type} {field1}, {Type} {field2}) {

    public {TypeName} {
        if (!{validationCondition}) {
            throw new IllegalArgumentException(
                "{clear message}: " + {field1});
        }
    }

    /** {Derived property description}. */
    public {Type} {derivedProperty}() {
        return {computation};
    }
}
```

## Sealed Result Template

```java
/** Outcome of {operation}. */
public sealed interface {Operation}Result
    permits {Operation}Result.Ok, {Operation}Result.Failed {

    record Ok({Type} {payload}) implements {Operation}Result {}
    record Failed(String reason, Throwable cause) implements {Operation}Result {}
}
```

## Domain Exception Template

```java
/** Raised when {domain condition}. */
public class {Domain}Exception extends RuntimeException {

    public {Domain}Exception(String message) {
        super(message);
    }

    public {Domain}Exception(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Service Class Template

```java
/** {Single responsibility of this service}. */
public class {Domain}Service {

    private static final Logger log =
        LoggerFactory.getLogger({Domain}Service.class);

    private final {Dependency} {dependency};

    public {Domain}Service({Dependency} {dependency}) {
        this.{dependency} = Objects.requireNonNull({dependency});
    }

    /** Main public entry — orchestrates the workflow. */
    public {Result} {mainMethod}({Input} input) {
        validate(input);

        var {stepOneResult} = {stepOne}(input);
        var {stepTwoResult} = {stepTwo}({stepOneResult});

        return {buildResult}({stepTwoResult});
    }

    private void validate({Input} input) { ... }

    private {Intermediate} {stepOne}({Input} input) { ... }

    private {Intermediate2} {stepTwo}({Intermediate} value) { ... }

    private {Result} {buildResult}({Intermediate2} value) { ... }
}
```

## CLI Command Template (picocli)

```java
// Main.java — bootstrap only, zero logic
public final class Main {
    public static void main(String[] args) {
        int exitCode = new CommandLine(new RootCommand()).execute(args);
        System.exit(exitCode);
    }
}
```

```java
// cli/RootCommand.java — picocli definitions only
@Command(
    name = "{cli-name}",
    mixinStandardHelpOptions = true,
    version = "{cli-name} {version}",
    description = "{Tool description — becomes --help header}.",
    subcommands = {ScanCommand.class, SyncCommand.class})
public class RootCommand {}
```

```java
// cli/ScanCommand.java — translate args, dispatch to runner
@Command(name = "scan", description = "{Command description}.")
public class ScanCommand implements Callable<Integer> {

    @Option(names = "--all", description = "Include repos with no remote.")
    boolean all;

    @Override
    public Integer call() {
        return new ScanRunner().run(new ScanRequest(all));
    }
}
```

```java
// commands/ScanRunner.java — interface layer: call service, format output
public class ScanRunner {

    public int run(ScanRequest request) {
        try {
            var report = new ScannerService().scan(request);
            Display.printReport(report);
            return 0;
        } catch (ScanException e) {
            Display.printError(e.getMessage());
            return 1;
        }
    }
}
```

## Display Class Template

```java
/** Semantic terminal output. Imported by commands/ only. */
public final class Display {

    private Display() {}

    public static void printSuccess(String message) {
        System.out.println(Ansi.AUTO.string("@|green ✓|@ " + message));
    }

    public static void printError(String message) {
        System.err.println(Ansi.AUTO.string("@|red ✗|@ " + message));
    }

    public static void printWarning(String message) {
        System.err.println(Ansi.AUTO.string("@|yellow ⚠|@ " + message));
    }
}
```

## Test Class Template (JUnit 6 + AssertJ)

```java
class {Domain}ServiceTest {

    @Test
    void {returnsExpectedWhenCondition}() {
        var {fixture} = new {Type}({fixtureValues});

        var result = {service}.{method}({fixture});

        assertThat(result).isEqualTo({expected});
    }

    @ParameterizedTest
    @CsvSource({
        "{input1}, {expected1}",
        "{input2}, {expected2}",
    })
    void {behaviorAcrossCases}({InputType} input, {OutputType} expected) {
        assertThat({function}(input)).isEqualTo(expected);
    }

    @Test
    void {throwsWhenInvalid}() {
        assertThatThrownBy(() -> {service}.{method}({invalidInput}))
            .isInstanceOf({Domain}Exception.class)
            .hasMessageContaining("{context}");
    }
}
```

## Concrete Example: Repo Scanner Service

```java
/** Repository discovery and dirty-state scanning. */
public class ScannerService {

    private static final int DEFAULT_MAX_DEPTH = 3;

    private final GitClient git;

    public ScannerService(GitClient git) {
        this.git = Objects.requireNonNull(git);
    }

    /**
     * Scans {@code root} for git repositories with uncommitted changes.
     *
     * @throws ScanException if {@code root} does not exist
     */
    public List<RepoEntry> scanDirtyRepos(Path root) {
        if (!Files.isDirectory(root)) {
            throw new ScanException("scan root not found: " + root);
        }

        return discoverRepos(root, DEFAULT_MAX_DEPTH).stream()
            .filter(repo -> git.hasUncommittedChanges(repo.path()))
            .toList();
    }

    private List<RepoEntry> discoverRepos(Path root, int maxDepth) { ... }
}
```
