# Prompt Templates

Modular prompt templates for different task categories and agent roles. Adapt these templates by populating them with concrete, verified facts from the codebase.

---

## 1. Implementation Plan Prompt (For Planning Agents)

Use when the user wants an agent to produce a structured delivery/implementation plan rather than executing the changes immediately.

```markdown
Create a detailed implementation plan for <objective/feature description>.

### Context & Requirements
- **Background**: <Why this change is needed, current behavior, and why the current approach is insufficient.>
- **Key Objectives**:
  - <Objective 1>
  - <Objective 2>
- **Constraints & Non-Goals**:
  - <Constraint 1: e.g. backward compatibility, suppressibility, performance>
  - <Non-Goal: what should NOT be touched or refactored>

### Technical Specifications
- **Target Modules / Packages**: `<module/package/path>`
- **Core Abstractions / Interfaces**: `<Class/Interface/Type names>`
- **Expected Precedents**: Follow the pattern established in `<existing/reference/file.kt>` for `<similar feature>`.

### Proposed Architecture & Code Changes
- <Component 1>: Describe planned changes, method overrides, or extensions in `<file/path>`.
- <Component 2>: Describe data structures or registration logic in `<file/path>`.
- <Generators/Tooling>: List any generation tasks required (e.g., `./gradlew <task>`).

### Test Impact & Verification Strategy
- **Anticipated Regressions / Affected Existing Tests**:
  - `<path/to/test1>`: Expected failure mode and required adjustment.
  - `<path/to/test2>`: Expected warning or suppression requirement.
- **New Test Coverage**:
  - Positive test cases validating expected behavior.
  - Negative test cases validating rejection/diagnostics.
  - Edge cases (nullability, type parameters, scoping, `@Suppress`).
- **Targeted Test Execution**:
  - `./gradlew <module>:test --tests "<fully.qualified.TestName>"`

### Key Reference Files
- `<path/to/file1>`: <One-line purpose>
- `<path/to/file2>`: <One-line purpose>
```

---

## 2. Compiler Diagnostic Prompt

Use when introducing a new compiler error, warning, or linter check (FIR, Frontend, or Backend).

```markdown
Implement a new <Compiler/Platform> diagnostic `<DIAGNOSTIC_NAME>` reporting a <WARNING/ERROR> when <trigger condition>.

### Context & Motivation
- **Current Behavior**: <Describe current permissiveness or existing diagnostic behavior.>
- **Target Behavior**: <Describe the new diagnostic, what it targets, and why.>
- **Severity**: `<WARNING | ERROR>`
- **Suppressibility**: Must be suppressible via `@Suppress("<DIAGNOSTIC_NAME>")` (or non-suppressible if mandatory error).
- **Positioning Strategy**: `<PositioningStrategyName>` anchored on `<AST/FIR/PSI node target>`.
- **Diagnostic Message**: Formulate the user-facing message in `<MessagesFile.kt>`, e.g., `"<Message text with placeholders {0}>"`.

### Checker Logic & Generator Steps
1. **Diagnostic Definition**:
   Add `<DIAGNOSTIC_NAME>` to `<path/to/DiagnosticsList.kt>` under the `<GROUP_NAME>` group.
2. **Message Registration**:
   Register default message in `<path/to/ErrorsDefaultMessages.kt>`.
3. **Regenerate Diagnostic Factories**:
   Run `./gradlew :<module>:generateDiagnostics` to update generated error classes.
4. **Checker Implementation**:
   In `<path/to/Checker.kt>`, inspect `<node.superTypes / expressions>` using `<helperMethod()>` and report `<DIAGNOSTIC_NAME>` on `declaration.source`.

### Test Suite Impact
- **Existing Tests Requiring Updates / Suppressions**:
  - `<path/to/existingTest1.kt>`: Emits new diagnostic, update testData or add `@Suppress`.
  - `<path/to/boxTest.kt>`: Codegen/box test that implements the restricted construct; update test expectations or suppress.
- **New Diagnostic Tests**:
  - Create `<path/to/newDiagnosticTest.kt>` validating:
    - Declaration types (classes, interfaces, objects, anonymous objects).
    - Expected diagnostic markers `<!<DIAGNOSTIC_NAME>!>...<!>`.
    - Verification that `@Suppress("<DIAGNOSTIC_NAME>")` successfully silences the diagnostic.

### Key Reference Files
- `<path/to/ExistingChecker.kt>`: Reference implementation of sibling diagnostic `<SIBLING_DIAGNOSTIC>`.
- `<path/to/DiagnosticsList.kt>`: Diagnostic registration DSL.
- `<path/to/ErrorsDefaultMessages.kt>`: Diagnostic message bundle.
- `<path/to/existingTestData.kt>`: Reference test data file.
```

---

## 3. Bugfix & Reproduction Prompt

Use when prompting an agent to fix a bug or issue reproduction.

```markdown
Investigate and resolve <Issue summary / KT-XXXXX>.

### Issue Description & Repro
- **Symptom**: <Describe unexpected behavior, crash, exception, or incorrect codegen.>
- **Reproduction Snippet**:
```kotlin
<Minimal reproducing code snippet>
```
- **Observed Result**: <Exception stack trace, wrong output, or compiler crash>
- **Expected Result**: <Correct output, clean compilation, or graceful error>

### Root Cause & Relevant Code Paths
- Suspected component: `<path/to/SourceFile.kt>` around `<methodOrClassName>`.
- Key mechanism: <Explain what fails during lowering, resolution, or type checking.>

### Requirements & Edge Cases
- Fix must not break <existing related behavior or performance>.
- Ensure proper handling of:
  - <Edge case 1: e.g. generic type substitution>
  - <Edge case 2: e.g. nullable types / smart casts>
  - <Edge case 3: e.g. cross-module / binary dependencies>

### Verification & Testing
1. Add new reproduction test under `<path/to/testData/...>`.
2. Run `./gradlew generateTests` to update test runners.
3. Run targeted test: `./gradlew :<module>:test --tests "<TestRunnerClass.testMethod>"`.
4. Run regression suite: `./gradlew :<module>:test --tests "<RelatedTestSuite>"`.

### Key Reference Files
- `<path/to/SourceFile.kt>`: Primary code location to fix.
- `<path/to/RelatedTest.kt>`: Existing tests for similar functionality.
```
