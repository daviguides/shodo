# Java Style Specification

## Formatting Tools

### Spotless + palantir-java-format
- **Format with Spotless** running **palantir-java-format** — 120
  columns, 4-space indent, lambda-friendly (the Java convention; do
  not impose the Python 80 here)
- The formatter owns line breaking and indentation — never
  hand-format
- `./mvnw spotless:apply` locally; `spotless:check` in CI

**Configuration pattern (`pom.xml`):**
```xml
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <configuration>
        <java>
            <palantirJavaFormat/>
            <removeUnusedImports/>
            <importOrder/>
        </java>
    </configuration>
</plugin>
```

### Error Prone + NullAway as Law
- **Error Prone on the compiler** — bug patterns fail the build, not
  the review
- **NullAway** (as an Error Prone plugin) enforces JSpecify
  `@NullMarked` nullness
- Suppress a check only with `@SuppressWarnings` at the smallest
  scope, WITH a comment stating why

## Naming Conventions

| Item | Convention |
|------|------------|
| Packages | `lowercase`, singular, no underscores |
| Classes, interfaces, records, enums | `PascalCase` |
| Methods, fields, variables | `camelCase` |
| Constants (`static final`) | `UPPER_SNAKE_CASE` |
| Type parameters | single capital (`T`, `E`) or `PascalCase` |

### Method Naming
- Verb-based: `calculateDiscount`, `validateEmail` — specific verbs
- Booleans in question form: `isActive()`, `hasRemote()`
- **No `get` prefix on records** — accessors are `name()`, not
  `getName()`; follow the record convention in adjacent code
- Conversion semantics: `toX()` copies, `asX()` wraps/views

### No Interface Noise
- No `IFoo`/`FooImpl` pairs — an interface with one production
  implementation is indirection, not abstraction; name the concrete
  thing (`GitClient`), extract an interface when a second
  implementation or a test seam demands it

## Class Organization

### Member Order
1. Constants
2. Fields
3. Constructors / static factories
4. Public methods (main entry first — orchestration reads top-down)
5. Private helpers, in call order
6. Nested types

### Size Limits
- One top-level type per file; file name = type name
- Class > ~300 lines or method > ~40 lines: split by responsibility
- More than ~4 constructor/method parameters: group into a record

```java
// ❌ Parameter creep
Report render(String text, Color color, boolean bold, int width)

// ✅ Options record with defaults via static factory
public record RenderOptions(Color color, boolean bold, int width) {
    public static RenderOptions defaults() {
        return new RenderOptions(Color.AUTO, false, 80);
    }
}
```

## Code Spacing

### Blank Lines
- 1 blank line between members
- 1 blank line between logical sections inside a method
- No blank padding at start/end of blocks

## Comments

### Strategic Comments
- **English only**, explain **why**, not what
- Prefer self-documenting names, records, and sealed types over
  comments
- `// TODO(context):` / `// FIXME(context):` for tracked debt
- Javadoc for API docs; `//` only for internal reasoning

## Annotation Hygiene

- `@Override` always — catches signature drift at compile time
- `@Nullable` (JSpecify) on every nullable return/field under
  `@NullMarked` packages
- `@Deprecated` always paired with Javadoc `@deprecated` explaining
  the replacement

## Consistency Enforcement

### Automated Tools
- `spotless:check` and Error Prone (with NullAway) in CI — no manual
  formatting, ever
- Checkstyle optional — only for rules Spotless cannot express
- Javadoc build (`mvn javadoc:javadoc`) must pass without warnings
