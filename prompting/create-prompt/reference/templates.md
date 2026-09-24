# WHAT-Brief Templates

One primary template plus three shape variants. Every variant describes **what must become true**,
never **how to make it true**. Populate them with verified repository vocabulary; omit any section
that has nothing real to say.

---

## 0. Primary Template (default — use unless a variant fits better)

```markdown
<One-sentence goal, phrased as an outcome, not an action on files>

### Background / Current Behaviour
<What the system does today, factually and verified against the repository, and why that is a
problem or a gap. Two to five sentences.>

### Desired Outcome
<What should be true once this is done, from the point of view of whoever is affected — user,
caller, build, operator. Outcomes, not steps.>

### Acceptance Criteria
- Given <situation>, <observable result>.
- Given <edge case>, <observable result>.
- <Invariant that must still hold afterwards.>

### Scope & Non-Goals
- In scope: <the boundary of this work>
- Out of scope: <explicitly excluded, so the planner does not expand the task>

### Constraints
- Hard: <must hold — compatibility, platform, severity, a technology the user genuinely mandates>
- Preference (non-binding): <the user's opinion, which the planner may overrule with reason>

### Open Questions for the Planner
- <Design decision deliberately left open.>
- <Unknown that recon could not settle.>

### Orientation
- `<subsystem or path>` — owns the behaviour in question.
- `<path>` — worth reading first.
- Prior art, reference only (not a pattern to copy): `<path>`.
- Verifiability surface: `<test suite or observable behaviour that would demonstrate the outcome>`.
```

---

## 1. Feature Variant

Use when something new should exist. Emphasis on the capability gap and on who benefits.

```markdown
<Capability that should exist, stated as an outcome>

### Background / Current Behaviour
<What people have to do today in the absence of this capability, and the cost of that.>

### Desired Outcome
<The new capability, described from the consumer's perspective.>

### Acceptance Criteria
- Given <consumer does X>, <the new capability produces Y>.
- Given <the feature is unused>, <existing behaviour is unchanged>.
- <Discoverability / documentation / error-reporting expectation, if it matters.>

### Scope & Non-Goals
- In scope: <the minimum coherent slice>
- Out of scope: <adjacent capabilities explicitly deferred>

### Constraints
- Hard: <compatibility, performance envelope, platform support, mandated technology>
- Preference (non-binding): <the user's leanings>

### Open Questions for the Planner
- <Naming, surface shape, defaults, or placement decisions left open.>

### Orientation
- `<subsystem>` — where this capability would live.
- Prior art, reference only: `<comparable existing feature>`.
- Verifiability surface: `<suite or manual check>`.
```

---

## 2. Bugfix Variant

Already WHAT-shaped by nature: symptom, reproduction, expected result. Keep it that way — do not
add a suspected root cause as an instruction.

```markdown
<Misbehaviour that should stop happening, stated as an outcome>

### Background / Current Behaviour
- **Symptom**: <what goes wrong, observably>
- **Reproduction**:
  <minimal steps or snippet>
- **Observed result**: <actual output, error, or stack trace>
- **Expected result**: <what should have happened>
- **Scope of impact**: <who or what is affected, how often, since when if known>

### Desired Outcome
<The reproduction produces the expected result, and nothing else regresses.>

### Acceptance Criteria
- Given the reproduction above, <expected result>.
- Given <related case that currently works>, behaviour is unchanged.
- The fix is covered by a regression check.

### Scope & Non-Goals
- In scope: <this misbehaviour>
- Out of scope: <related-but-separate issues, broader refactoring of the area>

### Constraints
- Hard: <no behavioural change for X; no performance regression; must work on Y>

### Open Questions for the Planner
- Root cause is not established — diagnosis is part of planning.
- <Whether a related edge case should be fixed in the same change.>

### Orientation
- `<subsystem>` — where the misbehaviour surfaces (not necessarily where it originates).
- Verifiability surface: `<existing suite covering this area>`.
```

---

## 3. Refactor Variant

The outcome is internal, so the invariants carry the weight: state what must **not** change.

```markdown
<Internal quality outcome, stated as a property the code should have>

### Background / Current Behaviour
<The concrete pain: what is hard to change, hard to understand, duplicated, or fragile today.
Give evidence, not an aesthetic complaint.>

### Desired Outcome
<The property the code should have afterwards, and what becomes easier as a result.>

### Acceptance Criteria
- External behaviour is unchanged: <how that is demonstrated>.
- <Observable structural property: e.g. a single place to change when X changes.>
- <Performance / build-time envelope preserved.>

### Invariants (must not change)
- <Public API surface / wire format / persisted data / observable timing.>

### Scope & Non-Goals
- In scope: <the specific area>
- Out of scope: <behaviour changes, adjacent cleanups, dependency upgrades>

### Constraints
- Hard: <must remain mechanically reviewable; no change bundled with feature work>
- Preference (non-binding): <direction the user leans toward>

### Open Questions for the Planner
- <Whether to do it in one change or a sequence.>
- <Which decomposition to adopt — deliberately left open.>

### Orientation
- `<path>` — the area in question.
- Verifiability surface: `<suite that pins current behaviour>`.
```

---

## Domain Appendix — Compiler Diagnostics (Kotlin)

A domain-specific slant for diagnostics work. Still WHAT-only: severity and suppressibility are
**requirements**, not implementation steps. Do not name checker files, registration DSLs,
positioning strategies, or generator tasks here — those are the planner's decisions.

```markdown
<Code pattern that should be reported, stated as an outcome>

### Background / Current Behaviour
<The pattern currently compiles silently; describe why that is harmful.>

### Desired Outcome
<Users writing this pattern are told about it at compile time, with an actionable message.>

### Acceptance Criteria
- Given <the offending pattern>, the compiler reports it at <the offending declaration/expression>.
- Given <a similar but legitimate pattern>, nothing is reported.
- The report is <suppressible / non-suppressible> as specified below.
- The message tells the user what to do instead.

### Scope & Non-Goals
- In scope: <the specific pattern and platform>
- Out of scope: <related patterns; changing existing diagnostics>

### Constraints
- Hard: severity is `<WARNING | ERROR>`.
- Hard: <suppressible via the diagnostic name / deliberately non-suppressible>.
- Hard: <source compatibility expectation for existing code>.

### Open Questions for the Planner
- Diagnostic name and exact message wording.
- Which frontend phase detects it, and how existing affected tests should be handled.

### Orientation
- `<subsystem>` — where this family of diagnostics is reported.
- Prior art, reference only: `<comparable existing diagnostic>`.
- Verifiability surface: `<the diagnostic test suite for this platform>`.
```
