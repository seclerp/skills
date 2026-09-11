# Prompt Pre-Flight Checklist

Evaluate the synthesized prompt against these 10 criteria before delivering it to the user.

---

### 1. Zero Implementation Leakage
- [ ] Has the current session refrained from modifying any project code, tests, or build scripts?
- [ ] Is the agent strictly delivering prompt content without attempting to run the planned work?

### 2. Zero Self-Planning Mode
- [ ] Is the agent avoiding creating task delivery plans or execution checklists for itself?
- [ ] Is the prompt addressed to the downstream recipient, with no self-directed task steps?

### 3. Concrete Repository Grounding
- [ ] Are all mentioned file paths verified to exist in the repository?
- [ ] Are paths written as repository-relative paths (e.g. `compiler/fir/...`) rather than absolute paths?
- [ ] Are class names, functions, diagnostic names, and interfaces verified against source definitions?

### 4. Established Precedents Identified
- [ ] Does the prompt reference at least one existing, working feature or diagnostic that serves as the architectural model?
- [ ] Are the similarities and deliberate differences between the precedent and the new work clearly stated?

### 5. Explicit Constraints & Non-Goals
- [ ] Are severity levels (`WARNING` vs `ERROR`) unambiguously stated?
- [ ] Is suppressibility (`@Suppress(...)` requirements or non-suppressible declarations) explicitly defined?
- [ ] Are non-goals or prohibited changes explicitly bounded so the downstream agent does not over-refactor?

### 6. Test Impact & Regression Assessment
- [ ] Did you search the test directories (`testData`, unit tests, box tests) for existing tests that use the construct being changed?
- [ ] Are existing tests that will fail due to unexpected warnings or errors explicitly listed by relative path?
- [ ] Does the prompt instruct how to handle affected tests (e.g., updating testData markers vs adding `@Suppress`)?

### 7. New Test Coverage Specification
- [ ] Does the prompt define what new tests must be written (positive cases, negative cases, edge cases)?
- [ ] Are edge cases specified (e.g., interfaces vs classes vs objects vs anonymous objects, type arguments, nullability)?

### 8. Generation & Build Tooling Grounded
- [ ] If code generation is required (e.g. diagnostic factories, test runners, protobuf), are the exact Gradle tasks provided?
- [ ] Are targeted test execution commands provided so the downstream agent can run fast, isolated tests?

### 9. Self-Containment
- [ ] Can a downstream agent successfully execute the task using ONLY the prompt, without access to the current conversational history?
- [ ] Are acronyms, issue links, and references clearly explained?

### 10. Clean Delivery Format
- [ ] Is all conversational preamble ("Sure, I have prepared...", "Here is the prompt for...") stripped out?
- [ ] If writing to a file (e.g., `prompt.md`), is the file formatted cleanly with no extraneous meta-commentary?
