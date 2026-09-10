# Directive Catalog

Full reference for header directives used by `kt-add-test`. Directives are declared as Kotlin objects extending
`SimpleDirectivesContainer`; syntax in testdata is `// DIRECTIVE` (simple) or `// DIRECTIVE: arg[, arg2]` (with
arguments). General mechanics: `compiler/test-infrastructure/ReadMe.md#directives`.

**Treat this file as a living document.** Directive names/behavior can change between Kotlin versions — if a directive
listed here doesn't exist anymore, re-grep the source file listed in its row before giving up on it.

## Classpath / runtime directives

| Directive | Source | Syntax | When to use |
|---|---|---|---|
| `WITH_STDLIB` | `ConfigurationDirectives` (`compiler/test-infrastructure/testFixtures/.../directives/ConfigurationDirectives.kt`) | `// WITH_STDLIB` | Snippet uses stdlib functions/collections (`listOf`, `println`, `map`, etc.) — without it, stdlib calls resolve to `UNRESOLVED_REFERENCE`. |
| `WITH_REFLECT` | `JvmEnvironmentConfigurationDirectives` (`compiler/tests-common-new/testFixtures/.../directives/JvmEnvironmentConfigurationDirectives.kt`) | `// WITH_REFLECT` | Snippet uses `kotlin.reflect`, `KClass`, `KType`, `::class` reflection APIs beyond simple class literals. |
| `WITH_KOTLIN_JVM_ANNOTATIONS` | `ConfigurationDirectives` | `// WITH_KOTLIN_JVM_ANNOTATIONS` | Snippet needs `kotlin-annotations-jvm.jar` (rare; JVM-specific annotations). |
| `STDLIB_JDK8` | `JvmEnvironmentConfigurationDirectives` | `// STDLIB_JDK8` | Snippet needs the JDK8 variant of stdlib. |
| `NO_RUNTIME` | `JvmEnvironmentConfigurationDirectives` | `// NO_RUNTIME` | Snippet must compile/run with no Kotlin runtime libs on the classpath at all. |
| `WITH_COROUTINES` | `AdditionalFilesDirectives` (`compiler/tests-common-new/testFixtures/.../directives/AdditionalFilesDirectives.kt`) | `// WITH_COROUTINES` | Snippet uses `suspend fun` / coroutine builders; adds coroutine-checking util functions. Add `WITH_STDLIB` too if it also calls real stdlib coroutine APIs (`kotlinx.coroutines`, `kotlin.coroutines`). |

## Backend directives

| Directive | Source | Syntax | When to use |
|---|---|---|---|
| `TARGET_BACKEND` | `ConfigurationDirectives` | `// TARGET_BACKEND: JVM_IR` | Test only makes sense on / should only run on the listed backend(s); test is skipped on all others. **If this is the only backend and it's JVM, the test must live in `codegen/boxJvm`, not `codegen/box`** (see `directory-map.md`). |
| `DONT_TARGET_EXACT_BACKEND` | `ConfigurationDirectives` | `// DONT_TARGET_EXACT_BACKEND: JS_IR` | Inverse of `TARGET_BACKEND` — skip the test only on the listed backend(s), run everywhere else. |
| `IGNORE_BACKEND` | `CodegenTestDirectives` (`compiler/tests-common-new/testFixtures/.../directives/CodegenTestDirectives.kt`) | `// IGNORE_BACKEND: JVM_IR` | Test is expected to compile/run everywhere but is a **known failure** on the listed backend(s) — failure there is tolerated instead of red. |
| `IGNORE_BACKEND_K2` | `CodegenTestDirectives` | `// IGNORE_BACKEND_K2: JVM_IR` | Same as `IGNORE_BACKEND` but only applies when the K2/FIR frontend is used. |
| `IGNORE_BACKEND_MULTI_MODULE` / `IGNORE_BACKEND_K2_MULTI_MODULE` | `CodegenTestDirectives` | same syntax | Multi-module variants of the two directives above. |
| `IGNORE_BACKEND_DIAGNOSTICS` | `CodegenTestDirectives` | `// IGNORE_BACKEND_DIAGNOSTICS` | Suppresses adding backend diagnostics to the metadata info handler — needed when running the backend on a test not originally designed for it. |

## Language / API version directives

| Directive | Source | Syntax | When to use |
|---|---|---|---|
| `LANGUAGE` | `LanguageSettingsDirectives` (`compiler/test-infrastructure/testFixtures/.../directives/LanguageSettingsDirectives.kt`) | `// LANGUAGE: +FeatureName -OtherFeature` / `// LANGUAGE: warn:FeatureName` | Snippet's behavior depends on a specific `LanguageFeature` enum entry being enabled (`+`), disabled (`-`), or enabled-with-warning (`warn:`). Feature names come from the `LanguageFeature` enum in `compiler/util/src/org/jetbrains/kotlin/config/LanguageVersionSettings.kt`. |
| `API_VERSION` | `LanguageSettingsDirectives` | `// API_VERSION: 1.9` | Snippet's behavior depends on `-api-version`, e.g. using a declaration annotated `@SinceKotlin(X)`. |
| `LANGUAGE_VERSION` | `LanguageSettingsDirectives` | `// LANGUAGE_VERSION: 2.0` | Rarely used directly — prefer `LANGUAGE` for individual features; only for whole-language-version pinning, and requires `ALLOW_DANGEROUS_LANGUAGE_VERSION_TESTING`. |
| `OPT_IN` | `LanguageSettingsDirectives` | `// OPT_IN: kotlin.RequiresOptIn` | Snippet uses an API that requires explicit opt-in. |

## Diagnostics-specific directives

| Directive | Source | Syntax | When to use |
|---|---|---|---|
| `DIAGNOSTICS` | documented in `compiler/testData/diagnostics/ReadMe.md` | `// DIAGNOSTICS: -WARNING +CAST_NEVER_SUCCEEDS` | Exclude/include specific diagnostics that would otherwise clutter the test (e.g. suppress `UNUSED_VARIABLE`). `+` = include, `-` = exclude, applied left to right. |
| `RUN_PIPELINE_TILL` | `TestPhaseDirectives` (`compiler/tests-common-new/testFixtures/.../directives/TestPhaseDirectives.kt`) | `// RUN_PIPELINE_TILL: FRONTEND` / `BACKEND` / `FIR2IR` | **Default to `BACKEND`** — the framework fails with "Phase FRONTEND could be promoted to BACKEND" whenever the pipeline could actually go further, and in practice most individual expected diagnostics (a single ambiguity, a single type mismatch on one expression, etc.) do **not** stop the rest of the file from reaching codegen, so `FRONTEND` is rejected even when diagnostics ARE expected — not just when zero are expected. Only use `FRONTEND`/`FIR2IR` when the file genuinely can't progress further (unresolved references, syntax errors, etc.); verify by trying `BACKEND` first. `FRONTEND` = FIR resolution only, `BACKEND` = through codegen, `FIR2IR` = fails once IR exists but before backend. |
| `CHECK_TYPE` | documented in `compiler/testData/diagnostics/ReadMe.md` | `// CHECK_TYPE` | Adds `checkType { _<Type>() }` helper declarations to assert exact expression types; also disables diagnostics related to `_` as a name. |
| `CHECK_TYPE_WITH_EXACT` | same | `// CHECK_TYPE_WITH_EXACT` | Like `CHECK_TYPE` but usable in codegen tests too (doesn't rely on disabling `_`-name diagnostics); needs explicit `checkExactType<T>(expr)` calls. |
| `RENDER_DIAGNOSTIC_ARGUMENTS` | `AGENTS.md` (`compiler/fir/analysis-tests/AGENTS.md`) | `// RENDER_DIAGNOSTIC_ARGUMENTS` | Needed if diagnostic markers include parenthesized arguments, e.g. `<!TYPE_MISMATCH("String; Nothing")!>`. |
| `RENDER_DIAGNOSTICS_FULL_TEXT` | `AGENTS.md` | `// RENDER_DIAGNOSTICS_FULL_TEXT` | Produces a companion `.fir.diag.txt` with human-readable error messages. |
| `DUMP_CFG` | `AGENTS.md` | `// DUMP_CFG` / `// DUMP_CFG: FLOW` | Generates a `.dot` control-flow-graph file — only if the ask is specifically about CFG shape. |
| `DUMP_INFERENCE_LOGS` | `AGENTS.md` | `// DUMP_INFERENCE_LOGS: FIXATION, MARKDOWN, MERMAID` | Only for inference-focused bug repros; dumps type-variable fixation/inference logs to companion files. |
| `LATEST_LV_DIFFERENCE` | `AGENTS.md` | `// LATEST_LV_DIFFERENCE` | Test expectations differ between stable and latest language version; requires a companion `.latestLV.kt` file (see `AGENTS.md`). |

## Bug-tracking directive

| Directive | Source | Syntax | When to use |
|---|---|---|---|
| `ISSUE` | `AGENTS.md` (`compiler/fir/analysis-tests/AGENTS.md`); used across all test kinds | `// ISSUE: KT-XXXXX` | Test is tied to a specific YouTrack bug report (from Step 2's issue-ID-only mode, or explicitly given by the user). |

## Structural directives (`FILE` / `MODULE`)

Full mechanics: `compiler/test-infrastructure/ReadMe.md#module-structure-directives`.

| Directive | Syntax | Notes |
|---|---|---|
| `FILE` | `// FILE: Name.kt` (or `.java`) | Everything until the next `FILE`/`MODULE` directive belongs to this file. Without any `FILE` directive, all content is one file named `main.kt`. |
| `MODULE` | `// MODULE: name` | Everything until the next `MODULE` directive belongs to this module. Without any `MODULE` directive, everything belongs to a default module named `main`. |
| `MODULE` with deps | `// MODULE: name(dep1, dep2)` | Declares module dependencies. |
| `MODULE` with friends | `// MODULE: name()(friend1, friend2)` | Empty `()` for deps when there are none but friends exist. |
| `MODULE` with dependsOn | `// MODULE: name()()(dependsOn dep1, dependsOn dep2)` | Empty `()()` for deps/friends when only `dependsOn` relations exist. |

Use `FILE`/`MODULE` for: multiple Kotlin files in one logical test, Java interop (a `.java` file alongside `.kt`), or
genuinely multi-module scenarios (e.g. testing `internal` visibility across module boundaries, or `expect`/`actual`).

## Auto-generated footers (never hand-write these)

- `/* GENERATED_FIR_TAGS: tag1, tag2, ... */` — appended automatically to diagnostics tests on their **first** run when
  absent; lists FIR constructs present in the file. If the first run of a fresh test only adds this footer and otherwise
  passes, that's expected tooling behavior, not a failure — see `AGENTS.md`.
- `.ir.txt`, `.kt.txt` (IR text tests), `.fir.txt` (FIR dump, only with `FIR_DUMP`) — all generated by the test run
  itself, never hand-written.
