# WHAT-ness Checklist

Run the brief against these gates before delivering it. Gates 1–4 are **rejecting**: if one fails,
rewrite the brief rather than patching around it.

---

### 1. No Solution Prescription (rejecting)

- [ ] Does the brief avoid step-by-step implementation logic?
- [ ] Does it avoid naming functions, classes, or signatures *to be added or modified*?
- [ ] Does it avoid "modify `<file>` so that it does X" instructions?
- [ ] Does it avoid choosing algorithms, data structures, or libraries on the planner's behalf?
- [ ] Does it avoid build, generator, and test commands to run?
- [ ] If a solution detail appears, is it under `Constraints`, labelled as a user-mandated
      constraint, rather than presented as a design?

### 2. Design Space Left Open (rejecting)

- [ ] Could two competent engineers read this brief, choose **different** designs, and both fully
      satisfy it? (Must be **yes**.)
- [ ] Are prior-art references clearly marked as *reference only*, not "follow this pattern"?

### 3. Verifiable Acceptance Criteria (rejecting)

- [ ] Is every criterion observable by someone who did not write the code?
- [ ] Is each one phrased as a situation and its result, not as a task to perform?
- [ ] Do the criteria cover the edge cases the user cared about?
- [ ] Do they state what must remain unchanged, where that matters?

### 4. Unknowns Surfaced, Not Invented (rejecting)

- [ ] Is every gap in the user's input either asked about or listed under
      `Open Questions for the Planner`?
- [ ] Is the brief free of requirements the user never actually stated?
- [ ] Are the user's non-binding preferences marked as preferences, not as hard constraints?
- [ ] Are any false premises in the user's input corrected rather than encoded?

### 5. Grounded Vocabulary

- [ ] Does every named path, module, subsystem, and symbol actually exist in the repository?
- [ ] Are paths repository-relative (e.g. `compiler/fir/...`), never absolute?
- [ ] Does the user's terminology map to real concepts, with corrections where it did not?

### 6. Clear Boundaries

- [ ] Is the in-scope boundary explicit?
- [ ] Are non-goals stated as boundaries of the ask, not as generic "don't over-refactor" advice?

### 7. Orientation Without Prescription

- [ ] Does the brief name useful starting points to read?
- [ ] Does it name a verifiability surface (suite or observable behaviour) without mapping expected
      test breakages or planning suppressions?

### 8. Self-Containment

- [ ] Can the planner work from the brief alone, with zero access to this conversation?
- [ ] Are acronyms, issue IDs, and external references explained or linked?

### 9. Length Discipline

- [ ] Is the brief within roughly 40–80 lines?
- [ ] Have empty or filler sections been omitted rather than padded?

### 10. Clean Delivery

- [ ] Is all conversational preamble stripped ("Sure, I have prepared…", "Here is the prompt…")?
- [ ] Is there no meta-commentary recapping the research steps?
- [ ] If written to a file, does the file contain only the brief?
