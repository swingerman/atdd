# Mutation Testing Frameworks Reference

Detailed setup, configuration, and usage for each supported framework.

> **Note:** Version numbers below are as of February 2026. Check each
> framework's website for the latest versions before installing.

---

## JavaScript / TypeScript — Stryker

**Website:** https://stryker-mutator.io/

### Install

```bash
npm init stryker@latest
```

Interactive wizard configures the project. Creates `stryker.config.mjs`.

### Configure

```javascript
// stryker.config.mjs
export default {
  mutate: [
    'src/**/*.ts',
    '!src/**/*.spec.ts',
    '!src/**/*.test.ts',
    '!generated-acceptance-tests/**',
    '!acceptance-pipeline/**'
  ],
  testRunner: 'jest',        // or 'mocha', 'karma', 'vitest'
  reporters: ['html', 'clear-text', 'progress'],
  coverageAnalysis: 'perTest',
  thresholds: { high: 90, low: 70, break: null }
};
```

### Run

```bash
npx stryker run
```

### Key flags

- `--concurrency 4` — parallel mutant workers
- `--logLevel trace` — debug output
- `--mutate "src/auth/**/*.ts"` — target specific files

### Mutation operators

Stryker supports: arithmetic, boolean, conditional, equality, logical,
string literal, array declaration, block statement, optional chaining,
and more.

---

## Python — mutmut

**Website:** https://github.com/boxed/mutmut

### Install

```bash
pip install mutmut
```

### Configure

```ini
# setup.cfg
[mutmut]
paths_to_mutate=src/
tests_dir=tests/
runner=python -m pytest -x
```

Or `pyproject.toml`:

```toml
[tool.mutmut]
paths_to_mutate = "src/"
tests_dir = "tests/"
runner = "python -m pytest -x --tb=no -q"
```

### Run

```bash
mutmut run
```

### Inspect results

```bash
mutmut results              # summary
mutmut show <id>            # show specific surviving mutant
mutmut html                 # generate HTML report
```

### Key flags

- `--paths-to-mutate src/auth/` — target specific directory
- `--runner "pytest -x"` — custom test runner
- `--use-coverage` — only mutate covered lines (faster)

---

## Java — PIT (pitest)

**Website:** https://pitest.org/

### Install (Maven)

```xml
<plugin>
  <groupId>org.pitest</groupId>
  <artifactId>pitest-maven</artifactId>
  <version>1.15.3</version>
  <configuration>
    <targetClasses>
      <param>com.example.*</param>
    </targetClasses>
    <targetTests>
      <param>com.example.*</param>
    </targetTests>
    <excludedClasses>
      <param>com.example.generated.*</param>
    </excludedClasses>
  </configuration>
</plugin>
```

### Install (Gradle)

```groovy
plugins {
    id 'info.solidsoft.pitest' version '1.15.0'
}

pitest {
    targetClasses = ['com.example.*']
    targetTests = ['com.example.*']
    excludedClasses = ['com.example.generated.*']
    mutators = ['DEFAULTS']
    outputFormats = ['HTML']
}
```

### Run

```bash
mvn pitest:mutationCoverage
# or
gradle pitest
```

### Mutation operators

PIT supports: conditionals boundary, increments, invert negatives,
math, negate conditionals, void method calls, return values, and more.

---

## C# — Stryker.NET

**Website:** https://stryker-mutator.io/docs/stryker-net/introduction/

### Install

```bash
dotnet tool install -g dotnet-stryker
```

### Configure

```json
// stryker-config.json
{
  "stryker-config": {
    "project": "MyApp.csproj",
    "test-projects": ["MyApp.Tests.csproj"],
    "mutate": [
      "src/**/*.cs",
      "!src/**/Generated/**"
    ],
    "reporters": ["html", "progress"],
    "thresholds": { "high": 90, "low": 70, "break": 0 }
  }
}
```

### Run

```bash
dotnet stryker
```

---

## Rust — cargo-mutants

**Website:** https://github.com/sourcefrog/cargo-mutants

### Install

```bash
cargo install cargo-mutants
```

### Run

```bash
cargo mutants
```

### Key flags

- `--file src/auth.rs` — target specific file
- `--jobs 4` — parallel workers
- `-d target/mutants` — output directory
- `--exclude "generated_*"` — exclude patterns

### Interpret results

```bash
cargo mutants --list          # preview mutations without running
```

---

## Go — go-mutesting

**Website:** https://github.com/zimmski/go-mutesting

### Install

```bash
go install github.com/zimmski/go-mutesting/cmd/go-mutesting@latest
```

### Run

```bash
go-mutesting ./...
```

### Key flags

- `--do-not-remove-tmp-folder` — keep mutations for inspection
- `--match "auth"` — target specific packages

---

## Ruby — mutant

**Website:** https://github.com/mbj/mutant

### Install

```bash
gem install mutant
# or add to Gemfile
gem 'mutant-rspec'
```

### Run

```bash
bundle exec mutant run --include lib --require my_app --use rspec 'MyApp*'
```

### Key flags

- `--since main` — only mutate changes since branch
- `--jobs 4` — parallel workers
- `--fail-fast` — stop on first surviving mutant

---

## Scala — Stryker4s

**Website:** https://stryker-mutator.io/docs/stryker4s/getting-started/

### Install (sbt)

```scala
// project/plugins.sbt
addSbtPlugin("io.stryker-mutator" % "sbt-stryker4s" % "0.16.1")
```

### Configure

```scala
// stryker4s.conf
stryker4s {
  mutate = ["src/main/scala/**/*.scala"]
  test-filter = ["com.example.*"]
}
```

### Run

```bash
sbt stryker
```

---

## Clojure — pitest via lein-pitest

### Install

```clojure
;; project.clj
:plugins [[lein-pitest "0.1.1"]]
:pitest {:target-classes ["empire.*"]
         :target-tests  ["empire.*-spec"]}
```

### Run

```bash
lein with-profile +pitest pitest
```

---

## General Configuration Tips

### Exclude patterns

Always exclude from mutation:

- `generated-acceptance-tests/` — generated pipeline output
- `acceptance-pipeline/` — parser/generator code
- Test files themselves
- Configuration files
- Migration files

### Performance

Mutation testing is slow. Optimization strategies:

1. **Use coverage data** — only mutate lines covered by tests
2. **Target specific files** — mutate changed files, not the whole project
3. **Incremental runs** — some frameworks cache previous results
4. **Parallel execution** — use available CPU cores

### CI Integration

For continuous integration, consider:

- Running full mutation testing nightly (slow)
- Running incremental mutations on PRs (fast, changed files only)
- Setting a threshold that fails the build if mutation score drops
