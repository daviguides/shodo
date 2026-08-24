# Java Testing Tools Specification

## Test Framework

### JUnit 6 (Jupiter Model)
- **JUnit 6** — unified versioning, Java 17+ baseline, JSpecify
  null-safety
- Test classes mirror the source package under `src/test/java`
- Names state behavior: `returnsErrorWhenConfigMissing`, not
  `testConfig2`; `@DisplayName` for prose when it adds clarity

```java
class ConfigLoaderTest {

    @Test
    void parsesValidConfig() {
        var config = ConfigLoader.parse(VALID_YAML);

        assertThat(config.maxRetries()).isEqualTo(3);
    }

    @Test
    void throwsWhenFileMissing() {
        assertThatThrownBy(() -> ConfigLoader.load(Path.of("missing")))
            .isInstanceOf(ConfigException.class)
            .hasMessageContaining("missing");
    }
}
```

## Assertions — AssertJ

- **AssertJ 3.x** for ALL assertions — never bare JUnit
  `assertEquals` (argument order errors, weak failure messages)
- `assertThat(...)` fluent chains; `assertThatThrownBy` for
  exceptions
- One logical assertion per test; chained checks on one subject
  count as one

## Parametrization

### @ParameterizedTest
```java
@ParameterizedTest
@CsvSource({
    "premium, 100, 90",
    "premium, 1000, 800",
    "standard, 100, 100",
})
void calculatesDiscount(String tier, long amount, long expected) {
    assertThat(Discounts.calculate(tier, amount)).isEqualTo(expected);
}
```

- `@CsvSource` for literal tables, `@MethodSource` for object cases,
  `@EnumSource` for exhaustive enum coverage
- Property-based note: **jqwik is in maintenance mode with a
  restrictive 1.10+ license clause** — prefer exhaustive
  parameterized tests; evaluate alternatives before adopting

## Mocking — Mockito

- **Mockito 5** — inline mock maker is default
- Mock **interfaces at seams you own** (a `GitClient` boundary), not
  records, not types you don't own beyond thin wrappers
- **Attach Mockito as `-javaagent`** in build config — dynamic agent
  self-attach warns on JDK 21+ and will be disallowed

```java
@Test
void skipsSyncOnMain() {
    GitClient git = mock();
    when(git.currentBranch()).thenReturn("main");

    var result = new SyncService(git).sync();

    assertThat(result).isInstanceOf(SyncResult.Skipped.class);
}
```

## Integration Tests — Testcontainers

- **Testcontainers 2.x** for real dependencies (Postgres, MongoDB,
  Kafka) — never in-memory fakes pretending to be the database
- Tag with `@Tag("integration")`; separate from unit runs

## Architecture Tests — ArchUnit

Layer rules from the CLI architecture spec become executable:

```java
@AnalyzeClasses(packages = "com.example.tool")
class ArchitectureTest {

    @ArchTest
    static final ArchRule servicesFreeOfCli =
        noClasses().that().resideInAPackage("..services..")
            .should().dependOnClassesThat()
            .resideInAPackage("picocli..");
}
```

## Async Assertions — Awaitility

```java
await().atMost(Duration.ofSeconds(5))
    .untilAsserted(() -> assertThat(queue.processed()).isTrue());
```

## Coverage — JaCoCo

```bash
./mvnw verify   # jacoco:report + check bound to verify
```

**Targets** (same bar as the Python spec):
- Overall: ≥ 90% (jacoco check `fail on < 0.90`)
- Business logic (services): 100%

Mutation testing with **PIT** on critical domain logic.

## Build Configuration Pattern

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <!-- compose agents: jacoco argLine + mockito agent -->
        <argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>
    </configuration>
</plugin>
```

```xml
<dependencies>
    <dependency>org.junit.jupiter:junit-jupiter (test)</dependency>
    <dependency>org.assertj:assertj-core (test)</dependency>
    <dependency>org.mockito:mockito-core (test)</dependency>
    <dependency>org.testcontainers:testcontainers (test)</dependency>
    <dependency>com.tngtech.archunit:archunit-junit5 (test)</dependency>
    <dependency>org.awaitility:awaitility (test)</dependency>
</dependencies>
```

## CI Commands

```bash
./mvnw spotless:check
./mvnw verify                      # compile (Error Prone) + tests + jacoco
./mvnw test -Dgroups=integration   # testcontainers suite
```

## Best Practices

- Arrange / act / assert separated by blank lines — no comments
  needed to mark the sections
- Test data minimal; builder/factory helpers over copy-paste
  fixtures
- Records make expected values trivial:
  `assertThat(result).isEqualTo(new Receipt(userId, 90))`
- Test the public seam of the module; package-private access is the
  Java equivalent of Rust's inline `mod tests`
- No test inheritance hierarchies — composition via helpers
