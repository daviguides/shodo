# Java Make Targets

Contract: `spec/make-contract-spec.md`. Use the build tool the
component already has; prefer the wrapper script committed in the
component (`./gradlew`, `./mvnw`) over a global install. Run offline
so the targets never download dependencies.

| Build | `test` | `check` |
|-------|--------|---------|
| Gradle | `./gradlew test --offline` | `./gradlew check -x test --offline` (lint, static analysis and format tasks wired into `check`, e.g. spotless, checkstyle) |
| Maven | `./mvnw -o -q test` | `./mvnw -o -q verify -DskipTests` (with the static-analysis plugins bound to `verify`) |

```make
.PHONY: test check format

test:
	./gradlew test --offline

check:
	./gradlew check -x test --offline

format:
	./gradlew spotlessApply --offline
```

Kotlin components follow the same targets; ktlint/detekt run inside
the build tool's `check`, never as separate global binaries.
