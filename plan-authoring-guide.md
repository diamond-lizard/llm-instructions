# Plan Authoring Guide

> **IMPORTANT: Read this entire document before authoring or editing implementation-plan documents. Following the principles here at authoring time produces concise, atomic, contextful plans that do not accumulate the kinds of problems that are expensive to fix later.**

This guide describes how to author implementation-plan Markdown documents in a way that produces clear, atomic, machine-and-human-friendly content the first time. The guide exists because rewriting an already-bundled plan to atomic form is expensive, error-prone work; getting it right at authoring time costs essentially nothing.

## 1. Why This Guide Exists

A plan document accumulates value over time as it is read, edited, and used by both humans and LLMs. Two failure modes degrade that value:

1. **Single bullet items, table rows, or paragraphs that bundle multiple distinct claims.** A bullet that reads "Implement X. Also implement Y. After Y, document Z. Then validate W." is not a single item — it's four items written as one. When such a bundle needs to be edited, cited, completed independently, or grepped, the bundle structure forces the editor to either split it on the spot (the slow, error-prone work) or accept that the four sub-items move together.

2. **Lines that are inconveniently long.** A line of, say, 2,000 characters is hard to read in a terminal, hard to diff cleanly, and tends to indicate the bundle problem above. But the inverse — chopping a line at an arbitrary character count — produces fragments that are themselves incoherent. The right fix is not to chop lines at column N; the right fix is to write atomic items.

This guide's central principle: **make every identifier-prefixed item, every bullet, every table row, and every paragraph carry one indivisible claim**. Long lines then either don't happen or are intrinsic to a single claim that genuinely cannot be expressed shorter.

## 2. The Atomic-Item Rule

Every identifier-prefixed item (`TASK-NNNN`, `GOAL-NNNN`, `REQ-NNNN`, `SEC-NNNN`, `CON-NNNN`, `GUD-NNNN`, `PAT-NNNN`, `ALT-NNNN`, `DEP-NNNN`, `FILE-NNNN`, `RISK-NNNN`, `ASSUMPTION-NNNN`) MUST express exactly one indivisible unit of its kind.

- A TASK is **one** action, with as much detail as needed to make that action unambiguous.
- A GOAL is **one** declarative statement of intent.
- A REQ is **one** mandatory property of the system.
- A SEC is **one** security control.
- A CON is **one** constraint.
- A GUD is **one** guideline.
- A PAT is **one** settled design pattern.
- An ALT is **one** rejected alternative.
- A DEP is **one** dependency.
- A FILE is **one** file.
- A RISK is **one** identified risk.
- An ASSUMPTION is **one** assumption.

**The atomic test.** Before assigning an identifier, ask:

- Could a reader understand this item without reading any other item?
- Could a future reviewer mark this item "done" or "addressed" independently of any other?
- Could this item be cited from elsewhere ("per TASK-NNNN") without ambiguity about which sub-claim is being cited?

If any answer is no, the item is bundled. Decompose it before assigning identifiers.

**The bundling test.** Conversely, before authoring an item, ask:

- Does the prose contain "and also", "then", "additionally", "furthermore", "after that"?
- Does the prose contain semicolons separating independent clauses?
- Does the prose enumerate multiple distinct cases (Case 1, Case 2, ...)?
- Does the prose enumerate multiple distinct branches of a state machine?

If yes, the item is at high risk of being a bundle. The right fix depends on what the enumeration is:

1. If the enumeration is multiple separate things to do (multiple validations, multiple implementations, multiple branches of a state machine where each branch is its own implementation step), decompose into multiple atomic sibling items — each gets its own identifier.
2. If the enumeration is multiple inseparable aspects of one indivisible action (the fields of a single data structure, the arguments to one function call, the rationale points behind one decision), move the enumeration into a Referenced Lists section (see Section 5) and have the atomic item reference it.

See Section 5 for the precise distinction. State-machine branches that are individually-implementable steps fall under case 1, not case 2.

## 3. Subject-Context Preservation

Every atomic item must restate enough subject context to be understood in isolation. This is the most-violated rule when a bundle is decomposed badly.

**Wrong (decomposed but ambiguous):**
- TASK-0100: Define the structure.
- TASK-0200: Populate the structure.

What structure? The reader has to scroll backwards to learn that the previous TASK introduced "the per-connection plist."

**Right (decomposed with subject restated):**
- TASK-0100: Define the per-connection plist's key set.
- TASK-0200: Populate the per-connection plist on connection accept.

Each item names what it operates on.

**The elision trap.** When you write a paragraph like:

> Produce the README. The README is the canonical install document. The README captures the operational contract once. The README must be self-contained.

It is tempting to decompose into:
- GOAL-0100: Produce the README.
- GOAL-0200: Be the canonical install document.
- GOAL-0300: Capture the operational contract once.
- GOAL-0400: Be self-contained.

GOAL-0200, GOAL-0300, GOAL-0400 are all broken: they don't say what the subject is. Fix:

- GOAL-0100: Produce the README.
- GOAL-0200: Make the README the canonical install document.
- GOAL-0300: Have the README capture the operational contract once.
- GOAL-0400: Make the README self-contained.

Even if the items are immediately consecutive, restate the subject on each one. Lists get reordered; items get cited from far away; LLMs read items individually.

## 4. Cross-Reference Discipline

Every cross-reference must point at the most-specific item the citing context actually means.

- "per TASK-NNNN" should point at the single TASK whose content the citing line is referring to. If the citing line genuinely refers to "the start command" as a whole, and "the start command" is now a bundle of TASK-0100 through TASK-0500, the correct citation is "TASK-0100 through TASK-0500" — not just "TASK-0300" arbitrarily picked from the middle.
- If you find yourself wanting to cite "the X" without an identifier, that is a signal that X is a bundle and should be decomposed (or that X is a properly atomic concept that simply hasn't been given an identifier yet).
- Citations to bundles of items use the inclusive range syntax: `TASK-0100 through TASK-0500`.

**No dangling citations.** If you remove or rename an item, every citation must be updated in the same edit.

**Citation rewrite procedure (when an item is removed, renamed, or split):**

1. Before making the edit, search the entire document for every occurrence of the old identifier. This is the citation manifest.
2. Apply the edit (removal, rename, or split).
3. For each citation in the manifest, re-read the citing line in full context and decide which new identifier(s) the citation should now point at. The decision is per-citation, not blanket. The mapping may be one-to-one, one-to-some, or one-to-all depending on what the citing line actually means.
4. Apply each rewrite.
5. Verify completeness: re-search the document for the old identifier. The count of unrewritten citations must be zero, EXCEPT in one case: when the original is split into multiple new sibling identifiers and one of those new siblings is allowed to reuse the original's number, the search will return that occurrence as the new sibling's own self-mention. Inspect each remaining occurrence in context and classify it as either (a) the new sibling's self-mention (a row or bullet whose identifier prefix matches the searched number), (b) a citation correctly rewritten to that same-numbered new sibling, or (c) a citation that was missed and still needs rewriting. Only category (c) is an error.

This procedure is mandatory whenever an identifier disappears from the document.

## 5. The Referenced Lists Mechanism

When an item's content includes a list, table, enumeration, sequence of steps, or other structured detail that doesn't fit one sentence, do not inline that structure into the item's prose. Move the structure to a top-level "Referenced Lists" section at the end of the document, and have the item reference it by quoted heading.

**Use Referenced Lists from inception.** Apply this mechanism while authoring, not as a later fix. When you find yourself writing an item with an inline enumeration that would itself qualify as bundled content, immediately move the enumeration to a Referenced Lists sub-heading and have the item reference it. Done at authoring time, the item is concise from inception.

**Hierarchical numbering.** The Referenced Lists section is numbered using whatever top-level section number follows the document's existing sections (e.g., if the document already has sections 1 through 7, Referenced Lists becomes section 8 — `## 8. Referenced Lists`). Sub-headings inside it use hierarchical numbering: `### 8.1 ...`, `### 8.2 ...`, with deeper levels `#### 8.1.1 ...`. The hierarchy lets a reader see at a glance what belongs together.

**Citation form.** The item references the sub-heading by quoting its full text including the hierarchical number prefix:

> ... per "N.M Descriptive Heading" below.

(The quoted heading text is schematic; substitute the actual hierarchical number and heading from your Referenced Lists section. For example, an item citing the section "8.4 Subprocess sentinel branching" would write `... per "8.4 Subprocess sentinel branching" below.`)

This is a stable handle: the heading text is unique; the number prefix groups related sub-headings; the citation is greppable.

**When to extract a list rather than decompose into siblings.** Two cases distinguish:

- *Aspects/parts of one indivisible action.* If the structured detail enumerates the fields of a single data structure, the arguments to one function call, the steps of a single algorithm whose steps are not independently executable, or the rationale points behind one decision, extraction is the right tool: the surrounding item stays atomic, and the detail moves to a referenced list.
- *Separate things to do.* If the structured detail enumerates multiple validations, multiple implementations, multiple goals, or multiple branches of a state machine where each branch is its own implementation step, decompose into siblings — each gets its own identifier. (Branches of a state machine are individually-implementable steps; do not extract them to a referenced list as if they were inseparable aspects of one thing.)

Use judgment per-case. The test is: would a reader marking this plan progress want to mark each piece of the structure done independently? If yes, sibling decomposition. If no, extraction.

## 6. Item Length Discipline

The guideline is: in implementation-plan documents, every physical line should be roughly under 300 characters. This applies to identifier-prefixed items, plain prose paragraphs, bullets in the Referenced Lists section, table rows — every kind of line in the plan. (The rule does NOT govern this guide's own physical lines or other explanatory documents; it is a property of plans, not of all Markdown.) This is a target, not a hard ceiling. The rule exists because:

- Long lines tend to be bundles (test: re-read with the bundling test in Section 2). The bundling test applies to identifier-prefixed items, prose paragraphs, and Referenced Lists bullets alike.
- Long identifier-prefixed items tend to embed enumerations that should be referenced lists; long Referenced Lists prose tends to bundle multiple separable claims into one paragraph (test: count semicolons or sentences that separate independent claims; if more than 2 in a single line, suspect inline enumeration).
- Long lines are awkward to read in terminals and to diff.

If a line is over 300 characters AND its content passes the atomic test AND has no extractable enumeration, leave it alone. Don't chop it at column 300; that produces incoherence. The 300-character target is a heuristic that points at decomposition or extraction opportunities; it is not a structural rule on its own.

## 7. Identifier Numbering Discipline

Use a consistent gap (typically 100) between identifiers in each family at authoring time:

- TASK-0100, TASK-0200, TASK-0300, ...
- GOAL-0100, GOAL-0200, ...

Reserve between-anchor numbers (e.g., `TASK-0150`, `TASK-0175` between `TASK-0100` and `TASK-0200`) for *insertions* during later editing. At authoring time, default to gap-100 spacing.

**Digit-width consistency.** Each identifier family uses a fixed digit width within a single document. Within a family, all identifiers share the same width: `TASK-00100` and `TASK-00200` (consistent), or `TASK-100` and `TASK-200` (consistent), but never `TASK-100` and `TASK-00200` (inconsistent). Different families in the same document may use different widths from each other. When extending an existing document, observe and preserve the width that family already uses. When authoring a new document from scratch, choose a width large enough for the expected family size (4 digits supports up to 99 items at gap-100; 5 digits supports up to 999 items at gap-100).

**Renumbering when identifiers drift.** After many splits and insertions, identifier numbering may no longer follow the gap-100 invariant. Renumbering — rewriting all identifiers in a family in document order to restore evenly-spaced gap-100 numbering, while updating every citation in the same pass — restores cleanliness for the next round of work. Renumbering can be done by hand (slow and error-prone for plans with many identifiers) or with a tool that processes both definitions and citations together.

**Renumbering risks.** Renumbering a mature plan has costs:

- Identifiers persistently referenced from outside the plan (commits, reviews, tickets, other documents) become stale. Do NOT renumber after identifiers have been externally cited unless every external reference can also be updated in the same change.
- Renumbering only renormalizes identifiers within the file being processed; references in other files cannot be updated automatically by an in-file renumbering pass.
- Renumbering may collapse digit width if a family's max index drops (e.g., `TASK-00100` may become `TASK-100` if the family no longer needs five digits). If a stable digit width is required, decide on the width policy before renumbering.

**Workflow when renumbering is desired.** Always renumber a copy of the file, diff the result against the original, confirm every renumbering is intentional, and only then adopt the renumbered copy as the canonical plan.

**Avoid identifier collisions with existing items.** When you choose a new identifier, do not pick a number that is already in use by an unrelated item. Either renumber the family to reflow it before adding new items, or pick a number not in use.

## 8. Bullet Lists vs. Tables vs. Prose

Use the right structure for each kind of content:

- **Identifier-prefixed bullet list** (`- **PREFIX-NNNN**: ...`) for top-level requirements/constraints/guidelines/risks/etc. — items that will be cited from elsewhere and don't have ancillary metadata columns.
- **Identifier-prefixed table row** (`| TASK-NNNN | ... | Completed | Date |`) for items that have ancillary metadata columns (status, completion date, owner). The table row form is heavier; reserve it for items that genuinely need columns.
- **Plain prose paragraph** for narrative context: phase introductions, document overviews, design rationale that does not need to be cited from elsewhere.

A common authoring failure: writing narrative prose into a TASK row's description, when the prose is actually phase-level context. Move it to a paragraph above the phase's task table; only put per-task content into table rows.

**Table-row constraints.** Markdown tables impose structural rules that authors must respect:

- Each table row must be exactly one physical line. Do not insert line breaks inside a row.
- Markdown does not natively support multi-line cells without HTML. If a cell's content needs more than one line, do not use HTML; instead, extract the content to a Referenced Lists sub-heading per Section 5 and have the cell reference it.
- Any literal pipe character (`|`) inside a cell must be escaped as `\|`. Inline code (backticks) does not change this — pipes inside backticked text still need escaping.
- A row's column count must match the header's column count. Do not omit trailing empty columns; use `|           |            |` (whitespace-only) for empty `Completed` and `Date` columns.

## 9. Phase Structure

A typical implementation-plan phase has this skeleton:

```
### Phase NNNN

- GOAL-NNNN: ...
- GOAL-NNNN: ...
- Local files read: ...
- Local files created: ...
- Local files modified: ...
- Local files deleted: ...

| Task     | Description           | Completed | Date       |
| -------- | --------------------- | --------- | ---------- |
| TASK-NNNN | ... |           |            |
| TASK-NNNN | ... |           |            |
```

- Each phase has its own GOAL identifiers (one or more, decomposed atomically per Section 2).
- The "Local files" bullets should each be one bullet. If a phase reads or creates many files, list them on separate bullets per file rather than packing many paths into one bullet.
- The TASK table holds per-task work items. Keep TASK descriptions atomic per Section 2. Long enumerations within a TASK move to the document's Referenced Lists section per Section 5.

**Multi-file Local-files example.** When a phase reads, creates, modifies, or deletes more than one file, repeat the category bullet per file rather than packing multiple paths into a comma-separated string. For example:

```
- Local files read: `path/to/PROTOCOL.md`
- Local files read: `path/to/firefox-to-emacs-native-messenger.el`
- Local files created: `path/to/firefox-to-emacs-native-messenger-wrapper`
- Local files modified: (none)
- Local files deleted: (none)
```

Each file becomes its own bullet under the appropriate category, repeating the category label. Categories with no files use the literal string `(none)`. This keeps each bullet under the line-length target and lets a reader grep for individual filenames.

## 10. Common Authoring Failure Modes

Recognize these patterns and fix them at authoring time:

- **The "and also" bundle.** Prose containing "and also", "additionally", "moreover" between independent claims. Fix: split into separate items.
- **The semicolon enumeration.** Prose using semicolons to separate distinct sub-actions. Fix: bullet list (extract to Referenced Lists if needed) or sibling decomposition.
- **The state-machine bundle.** Prose enumerating "if X then ... if Y then ... if Z then ..." across distinct branches of a state machine. Fix: one TASK per branch, with the branch condition restated.
- **The inline arg list.** Prose enumerating function arguments inline (`call F with :a 1, :b 2, :c 3, :d 4, ...`). Fix: extract to a Referenced Lists sub-heading.
- **The elliptical sub-item.** A sibling that begins with a verb but doesn't restate its subject. Fix: prefix the subject explicitly.
- **The inline rationale dump.** A REQ or PAT that includes detailed implementation rationale in parenthetical asides. Fix: extract rationale to Referenced Lists; keep the REQ/PAT prose narrow.
- **The orphan citation.** A citation `(per TASK-NNNN)` that, when followed, leads to a bundle whose actual relevant clause is hidden inside. Fix: at authoring time, decompose the cited TASK so the citation can point at exactly the right sibling.

## 11. The Authoring Checklist

Before declaring a plan section "drafted":

1. Re-read every identifier-prefixed item in isolation. Does each pass the atomic test (Section 2)?
2. For each long item, apply the bundling test. Decompose or extract as indicated.
3. For each item, confirm the subject is explicit (Section 3). Don't rely on the previous item's context.
4. For each cross-reference, confirm it points at the right specific item (Section 4).
5. For each enumeration inside an item, decide whether to keep it inline (only if 2 or fewer items AND each is atomic and short) or extract it to Referenced Lists (Section 5).
6. Confirm identifier numbering uses gap-100 spacing (Section 7).
7. Measure every physical line in the document. For any line over 300 characters, re-apply the relevant atomic, bundling, or extraction test per the line's type (identifier-prefixed item, prose paragraph, table row, Referenced Lists bullet).
8. Optionally renormalize identifier numbering to the gap-100 invariant before declaring the section drafted. Renumbering is appropriate for early-stage drafts and freshly-written documents, where the cost of renaming identifiers is low. Renumbering is NOT appropriate for mature plans whose identifiers are already cited from outside the document (commits, reviews, tickets, related files) — see Section 7's discussion of renumbering risks before deciding. When renumbering, work on a copy of the file, diff the result against the original, and confirm every renumbering is intentional before adopting the renumbered copy as the canonical plan.
9. Grep the document for each Referenced Lists heading text quoted in citations. Confirm exactly one heading line matches; confirm every citation's quoted heading text matches its target heading exactly. A mismatch indicates a stale citation or a renamed heading that must be reconciled.

A plan that passes this checklist will produce concise, atomic, contextful items that rarely need post-hoc remediation.

## 12. Worked Example

**Bad authoring (bundled, contextless, long):**

```markdown
| TASK-0700 | Implement the move handler. Run the path-expansion helper on both the from and to fields. Implement the upstream overwrite and cleanup semantics: if the destination is a directory, the effective target filename is the source's basename inside the destination directory; if the effective target exists and the request's overwrite field is not truthy, refuse with the corresponding response; otherwise rename via rename-file. After a successful rename, if the request's cleanup field is truthy, attempt to delete the original path. Build the response per PROTOCOL.md. |           |            |
```

**Good authoring (atomic, contextful, concise):**

```markdown
| TASK-0700 | Implement the move handler's path expansion. The move handler runs the path-expansion helper on both the from and to fields. |           |            |
| TASK-0800 | Implement the move handler's destination-is-directory normalization. If the destination is a directory, the effective target filename is the source's basename inside the destination directory. |           |            |
| TASK-0900 | Implement the move handler's overwrite refusal. If the effective target (from TASK-0800) exists and the request's overwrite field is not truthy, the move handler refuses with the corresponding response per PROTOCOL.md. |           |            |
| TASK-1000 | Implement the move handler's rename. When TASK-0900's overwrite refusal does not apply, the move handler renames the source to the effective target via rename-file. |           |            |
| TASK-1100 | Implement the move handler's cleanup branch. After a successful rename (TASK-1000), if the request's cleanup field is truthy, the move handler attempts to delete the original path. |           |            |
| TASK-1200 | Implement the move handler's response build. The move handler builds the response per PROTOCOL.md. |           |            |
```

Each row is short, atomic, contextful, and citable independently. The original bundle's content is fully preserved across the six new TASKs. Future editors can mark TASK-0900 done without affecting TASK-1100.

## 13. When Authoring Conflicts with Atomicity

In rare cases, an item genuinely is one indivisible claim and cannot be decomposed without loss of meaning. Examples:

- A REQ stating a single mandatory property, with one sentence of clarification.
- A CON stating a single environmental constraint.
- A FILE entry naming a single file with a one-paragraph description of its role.

Such items can legitimately exceed 300 characters if their indivisible content requires it. Do not chop them. The 300-character heuristic is a *suspicion-raiser*, not a *rule-enforcer*.

What you must NOT do: bundle multiple indivisible items into a single identifier just because they're related. "REQ-0100: The bridge MUST do X. The bridge MUST also do Y." is two REQs (REQ-0100 and REQ-0200), even if X and Y are tightly related.

## 14. Summary

- Every identifier = one indivisible item (Section 2).
- Every item restates its own subject (Section 3).
- Every cross-reference is specific, with a documented rewrite procedure when items disappear (Section 4).
- Long enumerations live in a Referenced Lists section, hierarchically numbered (Section 5).
- 300-character target is a heuristic, not a chopping rule (Section 6).
- Identifiers use gap-100 spacing, with consistent digit width (Section 7).
- Use the right structure (bullet, table row, paragraph) for each kind of content, with table-row constraints respected (Section 8).
- Phase skeleton is consistent (Section 9).
- Common failure modes are recognizable; fix them at authoring time (Section 10).
- Run the authoring checklist before declaring a section drafted (Section 11).

This guide aims to prevent bundled, long-line plans at authoring time so that no remediation work is needed.

## 15. Scope and Document Structure

This guide covers atomicity, subject-context, cross-reference discipline, length discipline, referenced-list extraction, and identifier numbering. It does NOT prescribe the full top-level structure of an implementation-plan document; it governs how each section's content is organized internally.

A typical implementation-plan document has the following top-level structure. Use it as a starting template; adjust per project as needed:

1. **Front matter (YAML).** Metadata: goal, version, dates, owner, status, tags.
2. **Introduction.** A few prose paragraphs naming the artifact being planned, the problem it solves, and the target environment.
3. **Requirements & Constraints.** Identifier-prefixed bullet lists for each requirement family: REQ (mandatory properties), SEC (security controls), CON (environmental constraints), GUD (guidelines), PAT (settled design patterns).
4. **Implementation Steps.** Phases (e.g., Phase 0100, Phase 0200, ...) each with their own GOAL items, "Local files" bullets, and a TASK table. See Section 9.
5. **Alternatives.** Identifier-prefixed bullet list of rejected alternatives (ALT items), each recording the alternative and the rationale for rejection.
6. **Dependencies.** Identifier-prefixed bullet list of external dependencies (DEP items): libraries, binaries, services, packaging requirements.
7. **Files.** Identifier-prefixed bullet list of files (FILE items) the plan produces, modifies, or interacts with, with absolute paths.
8. **Risks & Assumptions.** Identifier-prefixed bullet lists of identified risks (RISK items) and environmental assumptions (ASSUMPTION items).
9. **Related Specifications / Further Reading.** Plain bullet list of references: external specifications, upstream repositories, manual pages, related documents.
10. **Referenced Lists.** The extraction target described in Section 5. Holds long enumerations, sub-lists, or detailed sequences extracted from items elsewhere in the document. Hierarchically numbered (Section 5).

Apply the principles in this guide to every section's content, regardless of which top-level section it lives in.
