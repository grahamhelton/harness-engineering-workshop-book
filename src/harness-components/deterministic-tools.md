# Deterministic Tools

## Research task and learning outcome

- Maintain a source inventory and decide which changes need fresh review.
- Use a tool for a defined, repeatable question; let the researcher inspect its evidence and limits.
- A scanner match does not establish attacker control, reachability or impact.

## Concrete example: maintain a source inventory

- List direct Go `exec.Command`/`exec.CommandContext` calls under `util/`, excluding `*_test.go`.
- Report paths, lines and APIs; compare with the previous result.
- Not a complete command-injection review.

## Credible baseline and controlled change

- Proposed baseline: an ordinary agent-led inventory refresh with normal search and analysis tools available.
- Add the defined query, coverage checks and saved comparison; do not give that condition extra source or expected answers.
- Keep the question, source revisions, prior evidence, model, permissions and budget the same.
- The baseline may choose an analyzer too. A tie is valid; do not require the agent to imitate a parser badly.
- The earlier pilot below tested review routing, not whether a scanner beats all ordinary workflows.

## Demonstration outline

- Predict which source change matters. Preserve a baseline inventory and review decision.
- Repeat with the deterministic workflow; inspect coverage and matched expressions, not just the count.
- Introduce one counterexample, then decide whether the saved result still answers the original question.

## Parts of the implementation

- Rule: the saved question, such as a Semgrep pattern.
- Scan: answers for one source version.
- Saved comparison: changed facts and whether agent review is needed.
- Hypotheses/interpretation remain in [research state](memory/index.md#hypothesis--research-state).

## Detect useful changes

- Counts miss replacements; line numbers move with comments.
- Compare entries, keeping duplicates, by:

```text
(path, API, complete matched expression)
```

- Store line positions separately.
- Review first scans, changed expressions, removals and changed rules/tools/scope.
- Refresh line-only changes without claiming the code is secure.
- Block on missing evidence, scan errors, incomplete coverage or source changes during scanning.

## Improve the check

- Define files, patterns and exclusions.
- Start with established tooling; test should-match/should-not-match cases.
- Test ignore behavior and other call shapes, not only the motivating example.
- Save hashes, versions, raw results, coverage, prior state and limits.
- New rules change the question; repeatability is not completeness.

## Example coverage failure

- Semgrep's default ignores omitted `util/test/testutil.go` in the Argo pilot.
- Correction: enumerate tracked Go files, excluding test-suffix files; scan a source-only mirror.
- Override exclusions in the mirror, not the target; require expected/scanned paths to match.
- Keep a regression case in the missed file.
- Untracked files, indirect calls and other APIs remain outside scope; full file coverage is not full pattern coverage.

## Example interface

- Illustrative JSON—not an observed scan or supplied helper:

```json
{
  "analysis_status": "complete",
  "coverage_complete": true,
  "needs_review": false,
  "reason": "Same scope, tool, query and candidate expressions; only line positions changed"
}
```

- Analysis success and review decisions are separate.
- A routing layer must actually use the result; JSON alone does not change agent calls.
- Inventory source without executing target code.

## Counterexamples and evaluation

- Same-count argument replacement: unchanged counts can hide a changed expression.
- Line-only move: location changes need not require the same review as changed behavior.
- Normally ignored file: zero matches do not establish complete coverage.
- Also test unchanged source, unrelated document edits, additions, removals and changed query/tool/scope.
- Prepare independent expected inventories and review decisions. Score missed/extra entries, coverage gaps and unjustified skips.
- Record calls, elapsed time and available token/cache usage, plus scan/mirror preparation and rule maintenance.
- Record actual outcomes and skipped checks. No inventory CLI/replay lab is shipped here; these are an exercise design, not new results.

## Observed paired pilot and its limits

- Argo revision `fbd338cef5268b092489ed4a210c69c737d8c722`; equal task/model/source/prior inventory.
- Baseline: six agent sessions. Workflow with conditional review: four.
- All ten inventories accurate; source unchanged. Skipped review still refreshed line positions.
- Measured time was lower, but uncached input tokens increased.
- No token/cost savings or improved vulnerability-detection claim.
- One local pilot; human preparation excluded. Cache, latency, agent choices and change patterns affect results.

## Learner checkpoint and when not to add it

- Inspect an unfamiliar change. Give the inventory, coverage evidence, review decision, a limitation and one next action.
- Explain why an unchanged count is insufficient and why full file coverage is not full pattern coverage.
- A small one-off source question may need only ordinary search. A maintained rule must justify its setup and review costs.
- Unchanged matched expressions do not establish unchanged security behavior elsewhere.
- Repeat the proposed ordinary-workflow comparison before claiming broader accuracy, cost or teaching benefit.
