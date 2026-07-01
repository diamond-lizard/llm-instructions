# Long-Line Splitting Guide

> **IMPORTANT: Read this entire document from start to finish — every single line — before performing any line-splitting work. Every rule and technique described here is essential to producing a correct, faithful split.**

This guide describes how to reformat a Markdown document so that no line exceeds a target character length (typically ~300 characters), while preserving the document's meaning, names, cohesion, structure, and cross-reference integrity. The procedures here override any conflicting general-purpose editing intuition.

## 1. Scope and Applicability

This guide applies whenever a user asks for a long file's lines to be made shorter against a character target. It assumes:

- The file is Markdown.
- The file may contain numbered identifier families (e.g., `TASK-NNNN`, `GOAL-NNNN`, `REQ-NNNN`, `SEC-NNNN`, `CON-NNNN`, `GUD-NNNN`, `PAT-NNNN`, `ALT-NNNN`, `DEP-NNNN`, `FILE-NNNN`, `RISK-NNNN`, `ASSUMPTION-NNNN`).
- The file may contain Markdown tables, bullet lists, code spans (backticked content), and ordinary prose paragraphs.
- The user wants the work done with high care and zero loss of information.

If the target file does not match these assumptions, ask clarifying questions before proceeding.

## 2. Pre-Work Checklist (STOP AND READ)

Before making any edit, complete every step in this list:

1. **Confirm the target line length.** "Roughly 300 characters" is a soft target, not a hard ceiling. A line that lands at 320 may be acceptable if forcing it lower would harm cohesion. The threshold and tolerance are settled with the user up front; do not invent your own.
2. **Confirm whether the file is version-controlled.** If yes, use `git diff` for verification. If no, the user expects an in-place backup file (typically `<filename>.bak`) that you create before edits and overwrite after each verified iteration. Confirm this with the user.
3. **Confirm where extracted content (if any) should live.** Long content that cannot be split in place is moved to a new top-level "Referenced Lists" section at the end of the document, with descriptive sub-headings. Cross-references quote the heading text verbatim.
4. **Confirm the workflow cadence and approval mode.** Default cadence: split one line at a time, re-measure to find the new longest line, repeat until the target is met. Default approval mode: pause-for-approval before each split. The user may instead request autonomous mode (no pause), or batched mode (apply N splits, then verify). If autonomous, commit to extra care: errors will not be caught before they enter the file. Confirm both cadence and approval mode with the user before starting.
5. **Read the file fully** using your file-reading tool of choice before any edits. Knowing the document's structure end-to-end is a precondition for correct splits and cross-reference rewrites.

Skipping this checklist leads to wrong splits and broken cross-references that are very expensive to fix.

## 3. The Five Preservation Constraints

Every split must satisfy all five constraints. If any constraint cannot be satisfied for a particular line, do not split that line — record it as skipped with rationale.

1. **Meaning.** The cumulative semantic content of the resulting lines must be identical to the original line's meaning. No information is added or lost.
2. **Names verbatim.** Every function name, program name, file path, identifier, command name, configuration key name, error symbol, environment variable, library name, and quoted string must be reproduced exactly. Never split inside any of these.
3. **Cohesion.** If splitting a line would make for confusing reading, do not do it. Cohesion is a per-line judgment call: tightly-woven prose stays together; loose conjunctions of independent ideas are good split candidates.
4. **Cross-reference integrity.** When an identifier is split out of existence, every citation of that identifier elsewhere in the document must be rewritten to point at the correct new identifier(s) — see Section 7.
5. **Structural fidelity.** Markdown structures (headings, table rows, bullet lists, code blocks) must remain valid after the split. Do not break a table by inserting a non-row line into it; do not break a code fence; do not turn a heading into prose.

## 4. The Identifier Family Rule

Some lines start with an identifier from a fixed family, e.g., `- **TASK-09600**: ...`, `| TASK-09600 | ... |`, or `- GOAL-1200: ...`.

**Rule:** Identifier-prefixed lines may be split ONLY into other identifier-prefixed lines of the same family. They may not be split into bare prose paragraphs.

**Test for whether an identifier-prefixed line is split-able:**
- Read the line carefully. Is its content actually one indivisible unit (one task, one goal, one requirement), or is it a bundle of multiple distinct units that happens to be written in one prose paragraph?
- A bundle is something like: "produce the README; the README serves as the canonical install document; the README must allow a fresh reader to bring the bridge up; the README captures the operational contract; ..." — multiple discrete declarations, each independently meaningful.
- A genuine single unit is something like: "implement the cache directory verification function with the following exact behavior..." — one action, with supporting detail clarifying the same action.

**Subject-context preservation rule.** When you split a bundle into N siblings, each sibling MUST carry enough subject context to be understandable on its own. The bundle's prose typically establishes the subject early (e.g., "produce the README that..." establishes the README as the subject) and subsequent clauses may use elliptical references ("the README captures...", "captures the operational contract..."). When you extract one of those subsequent clauses into its own sibling, you MUST restore the elided subject. A sibling that reads "Capture the operational contract once, in the same directory as the source." is broken: it does not say WHAT captures the contract, nor WHICH source's directory. The fixed sibling reads "Capture the operational contract once, in the README, which lives alongside the bridge's source code in the project repository (FILE-0500)." Always re-read each new sibling in isolation; if you cannot understand the sibling without scrolling back to its peers, add the missing subject context.

**If the line is a bundle:** Split it. Each new identifier-prefixed line carries one of the discrete units. The original identifier disappears.

**If the line is a single unit:** Do not split it into multiple identifier-prefixed lines (that would create fake siblings). Instead, consider Section 6 (extract long content to a Referenced Lists section). If extraction is also infeasible, leave the line intact and record it as skipped.

**Critical mistake to avoid:** Never split an identifier-prefixed line into multiple bare paragraphs (with blank lines between them). That destroys the document's identifier structure and produces orphan prose that does not belong to any identifier family.

## 5. Numbering New Identifiers

When an identifier-prefixed line is split into multiple new identifier-prefixed lines:

- The original identifier disappears entirely from the document.
- New identifiers are integers, roughly evenly spaced between the immediately surrounding identifiers (the one before and the one after the original in the document).
- "Roughly evenly spaced" means: pick integers in the open range (lower-surrounding, upper-surrounding) such that the gaps between consecutive chosen numbers (and between each chosen number and the surrounding endpoint nearest it) are similar. Example: if the original was TASK-09600 and the surrounding identifiers are TASK-09500 and TASK-09700, and you produce three new TASKs, the cleanest evenly-spaced triple is 09550, 09600, 09650 (gaps of 50). For four new TASKs the cleanest quadruple is 09540, 09580, 09620, 09660 (gaps of 40). Numbers do not need to be round; 09533, 09600, 09666 is also acceptable.
- The new identifiers may use the same number as the original (e.g., 09600 appears as one of the new siblings even though it was the original's number). The original is considered "gone" because its meaning has been redistributed across the new siblings; the same-numbered new sibling is one of those new siblings, with its own new content. See Section 7 step 6 for how this affects post-split verification.
- If the original is the last in its family (no upper bound), use a reasonable upper bound that leaves room for future insertions.
- If the original is the first in its family (no lower bound), use a reasonable lower bound (typically the family's first identifier minus enough room for future insertions, but not zero or negative).
- The same numbering rule applies uniformly to every identifier family in the document.

**Resulting layout:** New identifier-prefixed lines appear as a tight block of consecutive lines at the position the original line occupied. There must be NO blank lines between them. Surrounding content (e.g., "Local files read", subsequent sections) remains in its original position relative to the block.

## 6. Extract-to-Referenced-Lists Mechanism

Some long lines describe a single semantic unit but contain a sub-clause that is itself a long enumeration, list, or self-contained body of detail. For these, the preferred technique is extraction.

**Procedure:**

1. Append a top-level section to the end of the document: `## N. Referenced Lists` (where N is the next available section number). Write a brief intro paragraph explaining its purpose.
2. Under that section, create a `### N.M Descriptive Heading` for each top-level extracted body, where N is the section number from step 1 and M is a sequential index (8.1, 8.2, 8.3, ...). Sub-bodies inside an extracted body use deeper levels (e.g., `#### N.M.K Descriptive Heading` for 8.1.1, 8.1.2, ...). The heading text combined with its hierarchical number is the long-term identity of the extracted material; it must be unambiguous, distinct, and grep-friendly.
3. Move the long content under that heading in any Markdown structure that suits it — sub-headings (`####`), bulleted lists, prose paragraphs, etc.
4. In the original line, replace the moved content with a short reference: `... per the section "N.M Descriptive Heading" below.` Quote the heading verbatim including its hierarchical number prefix, no Markdown link syntax.
5. Verify the heading text is unambiguous: it must occur in the document only at the heading line itself and at citation sites that quote it. Pick a heading text that is distinctive enough to satisfy this — generic words like "Notes" or "Details" do not satisfy it.

**When extraction is the right tool:**
- The extracted material is a list, table, or self-contained body of detail (multiple sub-items that share a common topic).
- The extraction reads naturally — the citing line still flows, and the extracted material reads as a stand-alone body when it is read.
- After extraction, the citing line is at or near the target length.

**When extraction is the wrong tool:**
- The extracted material is tightly woven into the surrounding sentence (a single phrase, not a list).
- Extracting would force the citing line to read awkwardly (e.g., the citing line becomes mostly cross-references).
- Extraction would not reduce the citing line meaningfully (e.g., the long part of the line is not the candidate-for-extraction part).

When extraction is the wrong tool and the line cannot be split into independent identifier-prefixed siblings either, leave the line intact and record it as skipped.

## 7. Cross-Reference Rewrite Procedure

Whenever an identifier disappears (because it was split), every citation of that identifier in the document must be rewritten. A dangling citation is a serious correctness violation.

**Procedure (perform immediately after each split, before the next split):**

1. **Before splitting, search the entire document for every occurrence of the old identifier.** This is the citation manifest. Record the line number and the citing context for each.
2. **Apply the split.**
3. **For each citation in the manifest, re-read the citing line in full context.** Decide which new identifier(s) the citation should now point at. The decision is per-citation and uses the full available context (the citing line's intent, the original identifier's meaning, the contents of each new identifier).
4. **The mapping may be one-to-one, one-to-some, or one-to-all:**
   - If the citing line refers to a specific aspect of the original that landed in exactly one new identifier, point at that one new identifier.
   - If the citing line refers to multiple specific aspects, point at the corresponding subset of new identifiers (which may be any subset, contiguous or not).
   - If the citing line refers to the original at the bundle level, point at all new identifiers, listed.
   - Use your judgment; do not blindly default to "one new identifier" or "all new identifiers."
5. **Apply each rewrite.**
6. **Verify completeness.** Re-search the document for the old identifier. The count of unrewritten citations must be zero. Note: per Section 5, one of the new sibling identifiers may use the same number as the original; in that case the search will return that occurrence as the new sibling's own self-mention, not as an unrewritten citation. Inspect each remaining occurrence in context to classify it as: (a) the new sibling's self-mention (a row or bullet whose identifier prefix matches the searched number), (b) a citation correctly rewritten to that same-numbered new sibling, or (c) a citation that was missed and still needs rewriting. Only category (c) is an error.

**Common pitfalls:**
- **Forgetting to search after the rewrite.** Always re-search.
- **Rewriting only the first occurrence in a multi-occurrence document.** A global search is required, not a single substitution.
- **Treating all citations the same.** Each citation deserves its own context-driven decision.
- **Rewriting too aggressively when the original is still present.** Check that the original truly disappeared before rewriting; some splits keep the original identifier alive on one of the resulting lines.

## 8. Identifier-Prefixed Markdown Table Rows

Some Markdown documents place identifier-prefixed items inside table rows (one row per item, with the identifier in the first cell). Such rows are long single physical lines containing pipes (`|`) as cell separators.

**Splitting rule:** A long identifier-prefixed table row is split into multiple consecutive table rows in the same table. Each new row is a complete row matching the table's column layout, with its own new identifier in the first cell. The new identifiers follow the numbering rule in Section 5 and the bundle-vs-single-unit test in Section 4. There are no blank lines between the new rows.

**Critical:** Do not introduce line breaks INSIDE a single table row. Markdown does not natively support multi-line cells without HTML, and most plan-style documents do not use HTML in cells. The split must produce multiple full rows, each with its own identifier.

**Cross-references to a split row's identifier follow the rules in Section 7.**

## 9. Unnumbered Prose Paragraphs

Plain Markdown paragraphs (text not prefixed by an identifier and not inside a list or table) may be split using either of these techniques, your choice based on what reads better:

- **Insert hard line breaks at sentence boundaries within a single paragraph.** The paragraph still renders as one paragraph in Markdown (since paragraphs are separated by blank lines, not single newlines).
- **Split into multiple paragraphs separated by blank lines.** Each resulting paragraph is its own unit; this works when the original paragraph contained multiple discrete topics.

**Do not** use the multi-paragraph approach on identifier-prefixed lines (see Section 4).

## 10. Skipping Decisions

Some lines cannot be split without violating one of the five preservation constraints. Skip those, with rationale recorded.

**Use your judgment.** Examples of natural skip candidates:

- The line is a single URL (no whitespace inside the long portion).
- The line is one indivisible quoted name or path.
- The line is a Markdown header.
- The line is a single semantically-cohesive sentence whose extraction targets are tightly woven into the surrounding prose.
- The line is one indivisible task/goal/requirement that has no extractable sub-content.

**Use your judgment per-line, not against rigid rules.** A judgment-based skip is better than a forced split that violates cohesion.

**Skip-list identity.** Each skipped line must be uniquely identifiable across iterations even though line numbers shift as the document is edited. Identify each skipped line by a fingerprint: an exact substring (long enough to be unambiguous in the document) that anchors the skipped line. The skip list is therefore a collection of fingerprints plus rationales, not a collection of line numbers. On every iteration, before picking the next longest line to address, search the file for each fingerprint to verify the skipped line is still present and to identify its current line number; the iteration ignores lines whose fingerprint is on the skip list.

Record every skip in the audit log with the fingerprint and a clear rationale.

## 11. Workflow

Apply this workflow strictly:

1. **Setup:**
   - Confirm the target length and tolerance.
   - Confirm backup approach (e.g., `<filename>.bak` for non-version-controlled files).
   - Confirm cadence and approval mode (one-at-a-time vs. batched; pause-for-approval vs. autonomous), per Section 2 item 4.
   - Read the entire file.
   - Review Section 16 to choose the right editing tool for each kind of edit you will make in this run.
2. **Make the initial backup.**
3. **Iterate until done:**
   - Measure all lines; pick the longest line strictly over the target that is not on the skip list.
   - Decide: split or skip.
     - If the line is identifier-prefixed: apply the bundle-vs-single-unit test from Section 4. If it is a bundle that decomposes into independent items of the same family: produce the new identifiers (Section 5), apply the split, then perform the cross-reference rewrite (Section 7). If it is a single unit, do not split into siblings — instead consider extraction (Section 6) or skip (Section 10).
     - If split via extraction (Section 6) ALONE (no identifier disappears): create the Referenced Lists section if it does not yet exist; add the new sub-heading; move the content; replace the moved content in the citing line with the heading reference. No cross-reference rewrites are needed because no identifier disappeared.
     - If extraction co-occurs with a sibling split (the original identifier disappears AND its content uses an extracted referenced list): perform the sibling split per Section 5, then perform the extraction for whichever new sibling needs it, then perform the cross-reference rewrites per Section 7 for the disappeared original.
     - If split as table rows: produce new rows (Section 8); cross-references (Section 7).
     - If split as prose paragraph: do the prose split (Section 9); no cross-references needed.
     - If skip: add the line to the skip list per Section 10 (record a fingerprint plus rationale).
   - Verify the iteration with a diff against the backup. The diff shows exactly what changed; confirm it matches your intent.
   - If the diff is correct, overwrite the backup with the current file state. The backup now represents the state before the next iteration.
   - Re-measure to find the next longest line.
4. **Final pass:**
   - Confirm no line exceeds the target except those on the skip list.
   - Re-search the document for every old identifier that should have disappeared. Apply the Section 7 step 6 classification: only unrewritten citations are errors; new-sibling self-mentions and correctly rewritten citations to a same-numbered new sibling are not.
   - Produce the audit report (splits made, skips with rationale, cross-reference rewrites).

## 12. Verification After Every Iteration

After every split or skip:

1. **Diff the file against the backup.** If non-version-controlled, this is `diff <filename>.bak <filename>`. If version-controlled, this is `git diff`.
2. **Read the diff carefully.** Every changed line should match the intent of the iteration.
3. **Re-measure all lines.** Confirm the iteration's split actually brought the targeted line under the threshold (if it was a split) or that the iteration left the line untouched (if it was a skip recorded in audit only). If a split produced one or more sub-lines that are themselves still over the threshold, those sub-lines are now in the queue for subsequent iterations.
4. **Re-search the document for any disappeared identifiers.** The count of unrewritten citations must be zero. If a new sibling happens to use the same number as the disappeared original (per Section 5), the search will return that new sibling's own self-mention; that occurrence is not an error. See Section 7 step 6 for the full classification of remaining occurrences.
5. **If anything is wrong, restore the backup and try again.** Do not patch a botched split with another split; restore and redo.
6. **If everything is right, refresh the backup with the current file state.**

## 13. Audit Trail

Maintain an audit trail throughout the work. The trail must include, for each iteration:

- The line number of the line being addressed.
- The line's character length.
- The action taken: split (with details) or skip (with rationale).
- New identifiers created, if any.
- Cross-references rewritten, with the mapping from old identifier to new identifier(s) and the citing locations.

The audit trail is the proof that the work was done correctly. Without it, neither you nor the user can reconstruct what was done.

## 14. Communication with the User

- Always use the user's preferred file-reading and file-editing tools (e.g., `ed` if the user's instructions require it).
- Always use absolute or repository-relative paths consistent with the user's explicit instruction; if the user has set a current working directory expectation, use relative paths from there.
- Always present multi-option choices with explicit option labels (Option A, Option B, etc.) so the user can answer with just the letter.
- Always ask one question at a time during clarification rounds; wait for the user's reply before continuing.
- Always honor the user's explicit instruction to do or not do something (e.g., "do not run git on this file," "do not use your memory tool without permission").
- Filter your option lists against constraints the user has already stated. Do not present an option that contradicts an existing instruction.

## 15. Common Failure Modes

- **Splitting an identifier-prefixed line into bare paragraphs.** Always produces orphan prose. See Section 4.
- **Forgetting cross-reference rewrites.** Always produces dangling citations. See Section 7.
- **Treating a single semantic unit as a bundle.** Produces fake sibling identifiers. See Section 4.
- **Treating a bundle as a single semantic unit.** Either produces a still-too-long line, or wrongly extracts material to a Referenced Lists section when sibling-splitting was the right tool.
- **Picking new identifiers that collide with existing siblings.** New identifiers must be strictly between the surrounding (existing, non-disappearing) identifiers in the family. Re-using the original-being-split's own number is allowed (per Section 5); re-using any OTHER existing identifier's number is a hard error. Always verify the chosen new numbers do not collide with any identifier other than the original.
- **Forgetting to verify after each iteration.** Errors compound silently.
- **Forgetting to refresh the backup after verification.** The next iteration's diff will show two iterations' worth of changes, masking the most recent one.

Read this guide before performing line-splitting work. The procedures here are designed to produce a correct, faithful, fully-audited revision; deviating from them is how splits go wrong.

## 16. Editing Tools and Long-Input Handling

This section describes which editing tool to use for each kind of edit. The decision tree below assumes the agent has the `ed` editor available (per the repository's `.github/ed-non-interactive-guide.md`), a `bash` tool with heredoc support, and a `create` file-creation tool.

### 16.1 Default tool

Use sync `ed` invoked via a quoted bash heredoc for every edit by default. Sync `ed` is the fastest tool and the one mandated by the ed editing guide.

### 16.2 When to use async `ed`

Use async `ed` (started via `bash mode="async"` and fed via `write_bash`) ONLY when the edit's content contains backtick characters. Per the ed editing guide's "MANDATORY: Interactive `ed` for Backtick Content" section, heredocs containing backticks are blocked by the shell security filter even inside single-quoted heredocs. Async `ed` bypasses this filter because the input goes directly to `ed`'s stdin with no shell interpretation.

Async `ed` is slower than sync `ed` because of the round-trip per write_bash call. Do not use it when sync `ed` would also work.

### 16.3 When to use the `create` tool

The bash heredoc input has a TTY-imposed line-length limit (approximately 4 KB per single source line). When you need to insert or restore a single document line whose character length exceeds this limit, sync `ed` via heredoc fails: the heredoc's content gets truncated by the TTY before reaching `ed`.

In that case, write the long line's content to a temporary file using the `create` tool (e.g., `/tmp/<descriptive-name>.txt`), then use sync `ed`'s `r` (read) command to read that file's content into the buffer at the chosen position. After the read, delete the temporary file. The `r` command reads the file contents as-is, so the long single line is inserted intact with no truncation.

### 16.4 Decision tree

```
Is the new content a single line longer than ~4 KB?
├── YES → Use the create tool to write a temp file, then ed `r` to read it in.
└── NO  → Does the content contain backticks?
         ├── YES → Use async `ed` via write_bash.
         └── NO  → Use sync `ed` via bash heredoc.
```

### 16.5 Verification after long-line edits

When the long-line content was inserted via the temp-file-and-read path, run a length check on the inserted line immediately after the edit using `bin/measure-line-lengths.py` (or equivalent). The measured length must equal the source content's length. If it differs, the read failed; restore the backup and retry. Delete the temp file only after the verification passes.

