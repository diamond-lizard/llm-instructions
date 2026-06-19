# Markup Conversion-Script Advice

General principles for writing a script that converts a structured or
semi-structured source document into a target markup format with formatting
constraints — and getting it right the first time. Ordered roughly by leverage.

## 1. Build the checks before the converter (test-first)
Write the checks before any transformation logic, and run them after every
change:

- **Format validator** — encode every rule of the target format as an assertion
  (structure, spacing, uniqueness, forbidden constructs, encoding). A
  hand-written validator can encode your *own* misreading of the spec, so, where
  one exists, also run the target format's own parser/linter, and include
  intentionally malformed fixtures to prove your validator actually rejects
  violations.
- **Coverage / fidelity check** — a token multiset alone is too weak: it misses
  reordering, title/body swaps, and duplicated or dropped repeated passages.
  Instead account for every source span — each must end in exactly one state:
  emitted verbatim, emitted after an allowlisted normalization, structurally
  converted, intentionally dropped as known boilerplate, or flagged ambiguous.
  Compare ordered text segments, and require every difference to map to an entry
  on the allowlist of intentional transformations.

Most defects in a conversion are *surfaced* by these checks rather than prevented
by design, so having them from the start turns silent bugs into immediate,
first-run failures.

## 2. Reconnoiter the source exhaustively — catalog its inconsistencies
Hand-authored sources are internally inconsistent. Before writing logic,
enumerate every structural construct **and each of its variations**, then build
a `(construct -> exact detection rule)` table and verify it against every
instance. Watch specifically for:

- inconsistent indentation for the "same" construct;
- a label that shares a line or paragraph with the content it introduces;
- names/titles wrapped across multiple lines;
- words hyphenated across a line break;
- irregular spacing (e.g., double spaces after a delimiter);
- embedded literal/art/tabular blocks (maps, tables, stat sheets, code);
- boilerplate lines (redundant titles, dividers) that must be dropped.

## 3. Parse into an intermediate representation; don't rewrite source straight to target
Detect source constructs into a tree/IR before emitting anything. Give each node
a type, its source span, normalized title/body fields, a literal-block flag, and
(when discarded) a reason. Validate the IR — containers present, levels correct,
coverage complete — and only then render the target markup from it. Separating
detection -> transformation -> validation -> rendering is what makes the other
principles enforceable; matching source text straight to output strings is
brittle and degrades fast as nesting deepens.

## 4. Detect by robust structural signals, combining positive and negative evidence
Prefer coarse, resilient signals — "indented vs. flush-left", "blank-line
separated", "all-caps line" — over exact whitespace counts or rigid regexes, but
don't let "coarse" become "ambiguous": confirm each classification with context,
precedence, and exclusion zones (e.g., "not inside a fenced block"), weighing
negative evidence as well as positive. Allow for punctuation variants (hyphens,
apostrophes, parentheses) inside labels and names. Assume your first detection
assumption is violated somewhere in the source.

## 5. Match the source's grouping granularity per construct
Grouping rules differ by construct, so decide them per construct rather than
globally. Blank-line-separated paragraphs are a common item boundary, but blank
lines also appear inside literal blocks, multi-paragraph items, and quoted text;
apply paragraph-wise parsing only after literal blocks are fenced (Principle 8)
and after confirming blank lines truly delimit *that* construct. Where they do,
the first line is the item head and the remainder is its body.

## 6. Maintain structural invariants
Emit from the IR (Principle 3) so each container exists exactly once — never have
several item-emitters independently produce a shared parent, which duplicates or
fragments sections. Assert the invariants directly: every node has its required
parent, no hierarchy level is skipped, and every required container is present.
These assertions catch missing, misplaced, or duplicated sections immediately.

## 7. Encode the target format's content rules explicitly — and test each
Translate each formatting rule into a concrete transform plus an assertion, for
example: uniqueness of sibling identifiers, consolidation/folding rules (short
content belongs in a title, not a one-line body), required spacing around
structural markers, and prohibited inline constructs. Don't discover these rules
reactively, one failure at a time. When a rule collides with verbatim
preservation — e.g., two siblings legitimately share a title — set an explicit
policy: keep the visible wording where possible, append the minimal structural
disambiguator only when forced to, and record every such collision for review.

## 8. Fence off opaque blocks before structural parsing
Identify embedded literal blocks (tables, diagrams, code/stat listings) by their
boundaries first and treat them as immutable payloads: structural detection,
line-wrapping, whitespace collapse, and token normalization must all skip their
interiors — only the enclosing markup (and any transformation the target format
explicitly mandates inside blocks) may change. Otherwise their internal
`key: value`-looking lines get misparsed as document structure.

## 9. Reflow conservatively, and treat join artifacts as ambiguous
Reflowing wrapped lines into a single field (e.g., folding into a heading title)
forces join decisions, each of which deviates from verbatim text, so scope them
tightly and resolve them conservatively. Collapse internal whitespace and strip a
leading space left after a delimiter, but treat an end-of-line hyphen as
*ambiguous*: it may be a real compound hyphen (keep it) or a soft line-break
hyphen (drop it). Only join when the result is unambiguous; preserve hyphens in
identifiers, all-caps terms, commands, and fenced blocks; and flag uncertain
cases. Keep body text verbatim (line breaks included) and normalize nothing you
are not actively reflowing.

## 10. Preserve wording by construction
Copy prose verbatim and restructure around it; never paraphrase. Only the format
changes — the words shouldn't. The coverage/fidelity check (Principle 1) enforces
this, and the disambiguation policy (Principle 7) bounds the rare, deliberate
exceptions.

## 11. Handle genuine ambiguity explicitly and surface it
Where the source is truly ambiguous, choose a deliberate rule, document it in
code, and report the decision for human review — rather than letting a generic
rule mishandle it silently. Likewise, handle each kind of boilerplate in exactly
one place.

---

**Meta-pattern:** Principle 2 (catalog the irregularities) prevents most defects
by design, while Principle 1 (test-first checks) surfaces the rest on the first
run. Everything else reduces to *parse into a validated tree, be tolerant in
detection, explicit and tested in rule enforcement, and verbatim in content.*
