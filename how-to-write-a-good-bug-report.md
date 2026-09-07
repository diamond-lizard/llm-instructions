# How to write a good bug report

A guide for drafting, reviewing, and submitting bug reports, incorporating this project's formatting preferences. Follow it for every bug report; do not re-derive the rules.

## Process: investigate before writing

A report is a *conclusion*, not an exploration. The investigation comes first, and its quality decides the report's quality.

1. **Pin the provenance of everything you test.** Confirm the artifact you read is the artifact that runs: check installed versions against source trees (read dist metadata, `version.py`, `git rev-parse HEAD`), and when comparing two versions, fingerprint files by hash rather than trusting file names or paths. A local venv labeled with a release number can be silently overwritten with development sources while only the dist metadata still names the release; every conclusion drawn from that tree would be wrong.
2. **Run experiments in isolation, with provenance assertions.** Point `sys.path` (or equivalent) at the exact tree under test and assert the imported module resolved from it. A probe whose path points at the package directory instead of its parent silently tests the wrong tree; identical results from supposedly different versions are the tell.
3. **Verify the bug against the real released artifact, not a stand-in.** When the bug is version-specific, download the actual release (wheel, sdist, tagged commit) and corroborate: the wheel and the sdist and the git tag should agree. Three independent witnesses beat one.
4. **Test the matrix, not just the repro.** If the suspected mechanism is "X happens when C", run all combinations of the confusable inputs. A matrix over two versions, two key-name spellings, and two candidate hashes can reveal a bug that is mirror-image destructive on both sides of a rename, which a single repro conceals.
5. **Expect the source issue's own description of the mechanism to be wrong.** A reported mechanism is a hypothesis. It can be wrong in its details while the bug is real: a range collapse blamed on a fallback branch may actually come from unknown keys silently dropped, so the range value never arrives. Verify claims against the artifact before restating them; a corrected mechanism is a finding.
6. **Trace to where the value is created, not where it fails.** The failing comparison may point at arithmetic that is actually correct; walking upstream to where the boundary value was assigned can find the real defect (a validation layer silently applying defaults after dropping unknown keys).

## Structure

Use this order; each part has one job.

1. **Title**: the symptom, not the presumed cause. "patch `range_hash` validated against the wrong line range" not "pydantic drops unknown keys". Causes get revised; symptoms survive.
2. **Summary**: one sentence a maintainer can act on: what input triggers what wrong behavior, and the scope (which versions).
3. **Consequences**: why anyone should care. Silent data loss, false rejections, trust erosion. Concrete user-visible outcomes, not internals.
4. **Detailed description**: the mechanism, code-level, with a reproduction and the evidence matrix. Claim per paragraph, linked per claim.
5. **Proposed solution**: ordered by invasiveness; say which subset you believe sufficient. Fix the mechanism (fail loudly on malformed input), not just the symptom.
6. **Environment**: OS, exact versions and refs tested. Nothing about the author's machine beyond that.

## Writing rules (project formatting preferences)

These are hard requirements, checked mechanically before submitting.

- **Every reference to source code, line or line ranges, and web pages is an inline markdown link.** No bare "see server.py" or "around line 400". Link text names the thing; the URL carries the precision. This covers every form of source reference: file names (link the file, e.g. [models.py](https://github.com/OWNER/REPO/blob/<sha>/src/models.py), even when the mention is inside backticks), commit SHAs (link only the SHA token, not the word commit, e.g. commit [395e663](https://github.com/OWNER/REPO/commit/<full-sha>)), git blob hashes (link to a file view at the commit carrying that hash), individual line numbers ([line 70](https://github.com/OWNER/REPO/blob/<sha>/src/server.py#L70)) and line ranges ([README L146-L154](https://github.com/OWNER/REPO/blob/<sha>/README.md#L146-L154)), in prose and in table cells alike. A reference is bare if a reader would have to search the repository to resolve it; linking the short SHA again, even when the full SHA appears elsewhere in the sentence, is the safe default.
- **GitHub source links are pinned to the full commit SHA** (`blob/<40-hex-sha>/...`), never a branch name: branches move, commits do not.
- **Verify cited line ranges against the pinned commit, and cite exactly.** Fetch the raw file at that commit and read the lines before submitting. A range must cover exactly the claimed code: extending it two lines past the comparison, or stopping one line short of the assignment that matters, is an error a maintainer will find and will discount the report for.
- **Never write that `main` is affected** unless verified which branch carries the bug; cite the default branch by its actual name in title, summary, and environment.
- **No em-dashes anywhere.** Use commas, colons, semicolons, or parentheses.
- **No backticks inside markdown link descriptions.** Backticks are for code outside links; a link description is prose naming the target.
- **Backtick identifiers; do not backtick English.** Code tokens get backticks outside links and fences: field names (`range_hash`), types and functions (`EditPatch`, `EditPatch.model_validate(p)`), commit SHAs, JSON literals (`"L2\n"`, `null` when it denotes the JSON value). Words that merely collide with an identifier stay plain ("closing the destructive path" is a route, not a field), as does prose like "reads lines 1..null" where the word means *nothing*. Quoted result values drop their quotes when they become code spans (`result "ok"` becomes result `ok`).
- **No bare version references.** "On 1.0.2" is ambiguous (latest release? the SDK? the server?). Write "PyPI 1.0.2", "PyPI 1.1.1", "mcp 1.25.0". This includes link descriptions: `[PyPI 1.1.1](...)`, not `[1.1.1](...)`.
- **Don't wrap long lines.** One paragraph, one physical line. Wrapping is for the reader's editor to soft-wrap; hard wraps render badly on GitHub and make diffs noisy. Do not stop reading a file because a line was truncated in your view: fetch the full line programmatically.
- **Nothing machine-specific** beyond general facts ("reproduced on Linux") and the exact versions/refs tested. Local paths, venv layouts, hostnames: leave them out; cite upstream repository paths and line numbers instead.
- **One question at a time** when a decision or clarification is needed during drafting; wait for the answer before continuing.

## Substantive preferences (learned in review)

- **Explain the circumstances, not just the mechanism.** "A patch keyed for the other version validates to the defaults" is incomplete until it says *when that happens and why it is not merely user error*: an undocumented rename while the release lags the code, patches generated from stale prompts or training data, mixed-version fleets, and documentation presenting both spellings. The reader should finish the paragraph able to recognize the situation in their own project, not just recall the mechanism.
- **Spell out why it is a server defect, not user error.** The facts that convert "caller mistake" into "reportable bug": the advertised schema promises to tolerate what it then silently mishandles (permissive schema, no `additionalProperties: false`); the safety mechanism designed to catch exactly this mistake is defeated by it (the hash validates the retargeted range instead of the requested one, and the caller's mistake would be recoverable if it were reported; it is destructive because it is silent); and internal inconsistency between related tools (different key spellings for the read path and the write path) makes even careful callers vulnerable.
- **State the expected result explicitly** in the reproduction: what the tool should return, what the file should look like afterward, and what constitutes a wrong outcome. "It broke" is not a reportable expectation; "the call returns `ok` and the file is truncated to one line" is.
- **Include the full evidence matrix as a table** when the bug is version-relative: rows for each input combination, columns per version, cell = outcome + whether the file survived. A matrix documents the mirror-image cases a single repro hides.
- **Corroborating corrections are findings.** If the source issue's mechanism is wrong, the report says so (politely, factually) and the corrected mechanism becomes part of the record.
- **Name the exact file or identifier whenever confusion is possible.** When a report discusses artifacts that share a name across versions or locations (a README on a branch and the same README shipped in releases, a config file in several packages, a symbol in two modules), never rely on a bare "the README" or "the config"; write which one is meant every time it could be ambiguous ("the README on the default branch develop", "the README shipped in PyPI 1.2.0") and link it. A scope note at the top of a section ("all references in this section are to X unless another release is named explicitly") can carry the disambiguation for a whole subsection, so individual sentences can say "the README" safely.
