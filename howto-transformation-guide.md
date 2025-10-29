# Howto Transformation Guide

## Objective
Centralize all actionable procedural instructions (how to perform tasks, operations, workflows) beneath a single heading titled `Howto`. Only the procedural instruction content is relocated; all other original sections, headings, and non‑procedural content remain in place and in their original order (except for the insertion of the new `Howto` section). Existing formatting conventions are preserved unless they directly conflict with practices required in this guide; in a direct conflict, the requirements here take precedence.

## Core Principles (Additions Clarifying Preservation)
1. Non‑procedural content (concepts, overviews, architecture, rationale, reference material, discussions) MUST be retained verbatim in its original sections.
2. Only actionable procedural instructions are moved in to the `Howto` section.
3. Original section ordering outside `Howto` is preserved; do not collapse, rename, reorder or delete unrelated sections unless necessary to resolve a direct conflict with this guide.
4. Existing formatting (heading casing, paragraph wrapping style, legitimate code blocks) is preserved unless it violates a required practice here (e.g., lists must be converted to headings; prompts removed from commands).
5. The Example Skeleton is illustrative only; it MUST NOT be treated as a required minimal final structure. Do not discard real sections that are not shown in the example.

## Scope of Content to Move
Include under `Howto`:
- Step-by-step instructions.
- Command sequences intended for execution.
- Configuration or data artifacts a user must create or edit.
- Operational variants and alternative methods.
- Troubleshooting or recovery sequences prescribing user actions.

Exclude from relocation (these stay where they are):
- Conceptual overviews, background, rationale, historical narrative.
- Descriptive prose without prescribed actions.
- Pure domain explanations and architectural descriptions.
- Reference enumerations (even if they mention actions) unless they are explicit “do this” procedural steps.

## Preservation & Relocation Strategy
1. Do not remove, summarize, or rewrite non-procedural sections.
2. If a mixed section contains both conceptual and procedural content, extract only the procedural segments to `Howto`; leave the conceptual portion intact in situ.
3. After extraction, if a source section becomes empty, you may delete that now-empty heading; otherwise retain it unchanged.
4. Maintain original heading levels and sequence outside the new `Howto` subtree.

## Transformation Procedure

### 1. Inventory and Extraction
1. Traverse the document top-to-bottom, tagging procedural fragments (paragraphs, list items, imperative sentences, command/output pairs, code+instruction clusters).
2. Record for each fragment:
   - Original location (heading path)
   - Exact text
   - Any inline numbering/prefixes
3. Leave the original document untouched until mappings are complete.

### 2. Normalize Procedural Units
1. Split multi-verb passages into separable units only if independence is clear.
2. Keep sequences intact if splitting harms coherence.
3. Deduplicate repeated procedures; retain the fullest authentic variant.

### 3. Assign Canonical Verbs
1. Map each procedural unit to a single base verb (e.g., Build, Configure, Deploy, Delete, Install, List, Monitor, Optimize, Provision, Recover, Restore, Scale, Schedule, Test, Update, Validate, Verify).
2. Choose consistent synonyms (e.g., select either `Delete` or `Remove`, not both).
3. Secondary actions (e.g., verification) may become sub-headings beneath the primary verb.

### 4. Insert the `Howto` Heading
1. Determine the correct depth (e.g., if there is a single top-level title, `Howto` is one level below it).
2. Insert `Howto` without blank lines before or after.
3. Ensure only one `Howto` heading exists.

### 5. Create Verb-Level Headings
1. Each canonical verb becomes a direct child under `Howto`.
2. Preserve meaningful original numeric/alphanumeric prefixes only if they carry external or semantic significance; otherwise omit.
3. Titles must be unique.

### 6. Add Method / Variant Sub-Headings
1. Under each verb heading, create sub-headings for distinct methods, contexts, deployment modes, platforms, or variants.
2. Further subdivide as needed (e.g., environment-specific, optimization variants).
3. Keep titles concise and action-driven.

### 7. Alphabetization (Recursive Under `Howto`)
1. Alphabetize verb-level headings (case-insensitive compare; retain original case).
2. Alphabetize immediate children of every heading under `Howto`.
3. Recurse depth-first; at each sibling level re-order entire blocks (heading plus its subtree).
4. If preserved literal prefixes exist, sort by prefix token first, then title text.

### 8. Convert Lists to Headings (Within Procedural Content)
1. Replace bullet/number lists representing steps with heading structures.
2. Retain numbering tokens only if they reflect meaningful source identifiers.
3. Do not simulate bullets with punctuation; rely exclusively on heading stars for structure.
4. Lists outside procedural content (e.g., conceptual enumerations) should be re-expressed as headings only if required by overarching conventions; otherwise preserve original formatting unless that formatting is disallowed.

### 9. Populate Each Procedural Heading
1. Provide explanatory prose immediately after the heading when needed (no blank line).
2. Pure container headings (organizational only) may have no body.
3. Remove redundant restatements.

### 10. Commands, Artifacts, and Output
1. Place commands and user-authored input artifacts in source blocks (no shell prompts).
2. Place authentic command output in immediately adjacent example blocks (no blank line separating the pair).
3. Do not intermingle output inside command blocks or vice versa.
4. Include full, verbatim output when available; omit the output block if no authentic output exists.
5. Never fabricate, redact, or truncate output; do not add ellipses or placeholders.
6. Avoid splitting a single coherent artifact across multiple blocks unless logically distinct segments exist.

### 11. Block Integrity
1. Use uppercase block delimiters.
2. Avoid blank first/last lines inside blocks; blocks must not be empty.
3. Use language tags only when adding clarity (lowercase tags).
4. Preserve meaningful internal spacing; remove only trailing whitespace and shell prompts.

### 12. Inline Formatting Restrictions
1. Remove inline emphasis or code markers for technical literals; move literals to blocks if needed.
2. Do not simulate code with quotes or capitalization.

### 13. Prose and Whitespace
1. Maintain the original wrapping style unless inconsistent or exceeding agreed readability width; reflow only where necessary.
2. Avoid consecutive blank lines.
3. Remove trailing whitespace.
4. Use spaces in prose; tabs only when semantically required inside blocks.

### 14. External Links
1. Use either bare URLs or explicit link syntax.
2. Provide meaningful labels when labeling links; avoid generic labels.

### 15. Numbering and Prefix Preservation
1. Preserve existing numeric/alphanumeric/dotted prefixes only if they are meaningful in source context.
2. Do not normalize or renumber for appearance.
3. Resolve prefix collisions among siblings by minimally adjusting (or by removing a non-essential prefix) while documenting the change if traceability is needed.

### 16. Remove Procedural Content from Original Locations
1. After relocation, delete only the procedural fragments that were moved, leaving non-procedural passages intact.
2. If a heading becomes empty post-removal, either:
   - Remove that heading, or
   - Re-purpose it with a clarifying retained conceptual description.
3. Do not remove or condense unrelated sections.

### 17. Cross-References and Internal Linking
1. When a procedure references a concept, term, or entity defined elsewhere, create an internal link from the `Howto` usage to the defining heading.
2. Use the canonical term as link text (shortest unambiguous form).
3. Avoid duplicating definitions inside `Howto`.
4. Prevent unnecessary reciprocal links that add noise.
5. Periodically scan for unlinked terms that have definitions.

### 18. Final Validation Checklist

Preservation:
- All original non-procedural sections remain (order intact).
- Only procedural fragments moved; nothing lost.

Structure:
- One `Howto` heading at correct depth.
- No heading level skips.
- All procedural content resides under `Howto`.

Organization:
- Verb-level headings under `Howto` alphabetized.
- All descendant sibling groups under `Howto` alphabetized.

Headings:
- No duplicate sibling titles.
- Preserved prefixes unique where used.

Procedural Content:
- Commands/artifacts in source blocks; no prompts.
- Authentic output in adjacent example blocks; full and verbatim.
- No fabricated or truncated output.

Formatting:
- No inline emphasis/code markers for literals.
- Prose formatting outside `Howto` preserved except where incompatible.
- No consecutive blank lines; no trailing whitespace.

Conversion:
- Original procedural lists replaced by heading structure.
- Meaningful numbering tokens retained; no cosmetic renumbering.

Linking:
- Cross-links from procedures to concept definitions added where terms appear.

Completeness:
- No orphaned procedural fragments remain outside `Howto`.
- Each heading either contains content or purposefully organizes child headings.

### 19. Suggested Verb Vocabulary (Optional)
Analyze, Backup, Build, Configure, Create, Deploy, Destroy, Diagnose, Export, Import, Install, List, Monitor, Optimize, Provision, Recover, Remove (or Delete—choose one), Restore, Scale, Schedule, Test, Update, Validate, Verify.

### 20. Example Skeleton (Illustrative Only — Do Not Delete Real Sections to Match)
```
* Project Guide
** Overview
*** Purpose
*** Audience
** Concepts
*** Architecture
*** Data flow
** Howto
*** Configure
**** Service file
#+BEGIN_SRC yaml
# configuration content
#+END_SRC
*** Deploy
**** Rolling method
#+BEGIN_SRC bash
# deployment commands
#+END_SRC
#+BEGIN_EXAMPLE
# authentic output
#+END_EXAMPLE
*** Verify
**** Health check
#+BEGIN_SRC bash
# verification commands
#+END_SRC
#+BEGIN_EXAMPLE
# authentic output
#+END_EXAMPLE
** Reference
*** CLI commands
*** Configuration keys
** Troubleshooting
*** Common errors
*** Performance issues
```
(Real documents may contain additional top-level or sibling sections not shown here; keep them.)

### 21. Automation Tips
1. Parse headings and build a structural map; mark procedural fragments by heuristic (imperative verbs, code blocks, pattern of “run”, “execute”, etc.).
2. Extract procedural nodes, classify to verbs using a controlled synonym map.
3. Generate the `Howto` subtree in memory; alphabetize recursively.
4. Replace original procedural fragments with nothing (or leave conceptual residue) while keeping surrounding content intact.
5. Insert cross-links by scanning `Howto` text for defined concept terms.
6. Run a linter enforcing adjacency of command/output blocks, heading uniqueness, absence of prompts, formatting hygiene, and preservation validation.

### 22. Quick Reference Checklist
- Preserve all non-procedural sections and their order.
- Move only procedural content into a single `Howto` hierarchy.
- Alphabetize siblings under `Howto` at every depth.
- Convert procedural lists to heading-based structure.
- Commands in source blocks (no prompts); outputs in adjacent example blocks, full & authentic.
- No fabrication/truncation/redaction of output.
- Maintain original formatting unless incompatible with required practices.
- Add cross-links from procedures to concept definitions.
- Retain meaningful numbering/prefixes; avoid cosmetic renumbering.
- Ensure clean whitespace and uniqueness of headings.
