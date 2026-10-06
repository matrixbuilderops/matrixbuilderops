# OmniCLI Orchestration and Red-Team Quality Plan

## Purpose and scope

OmniCLI is an orchestration and source-code red-team review tool. This plan
measures whether its orchestration behaves reliably and whether its reviews
produce useful, evidence-supported results. It is not a bug-bounty workflow,
and a benchmark pass is not proof that an arbitrary codebase is secure.

Keep two quality tracks separate:

1. **Orchestration reliability:** deterministic behavior such as routing,
   dependency order, bounded parallelism, failure propagation, resume, and
   resource-limit enforcement.
2. **Red-team review quality:** the correctness, evidence, coverage, and
   repeatability of model-assisted reviews on independently labeled code.

## Current baseline

- Offline suite: **194 tests and 51 subtests pass** after the current quality
  fixes.
- Development-only Python static baseline: **12/12 labeled positives and
  12/12 clean cases** on 24 self-authored cases. This is a regression smoke
  result only: those cases were used during development and are not independent
  evidence of general performance.
- Candidate holdout: **60 Python cases**, 30 positive and 30 benign across six
  categories. All remain provisional with one reviewer.
- A blinded second-reviewer packet has been prepared. Do not expose its private
  answer key before the independent review is returned.
- Prior provider pilot results were too small and failure-prone to establish
  efficacy. Provider errors must remain visible in future scores.

## Track A: deterministic orchestration checks

Keep these checks offline in CI with fake providers and controlled clocks:

- Routing selects the requested provider/model and respects configured
  family-diversity behavior.
- Independent DAG tasks overlap up to the configured concurrency limit;
  dependent tasks start only after prerequisites finish and receive the right
  outputs.
- A failed task is reported explicitly, its descendants are skipped, and
  independent branches continue.
- Checkpoint resume reuses successful work, retries only failed work, rejects
  incompatible run options, and does not duplicate completed side effects.
- Timeouts, cancellations, retry limits, and future run-wide token/cost/deadline
  budgets stop work predictably and appear in structured events.
- Partial provider outages never become success-shaped empty answers.

For each behavior, maintain at least one success case and one injected-failure
case. Record concurrency and event order in tests rather than relying on timing
impressions. Add workflow-level resume and run-wide budgets only when those
features are implemented; until then, document them as limitations.

## Track B: blinded red-team review benchmark

### Dataset and adjudication

- Use a Python-first dataset because that is the current labeled corpus and
  deterministic-analysis scope.
- Include vulnerable cases, confirmed clean cases, realistic near-misses,
  multi-file context, and cases where the correct answer is “insufficient
  evidence.”
- Record provenance, source revision, category/CWE where applicable, severity,
  exact evidence, preconditions, impact, and rationale for clean labels.
- Split by repository or source lineage, deduplicate near-identical samples,
  and keep development material out of the final holdout.
- Have a second reviewer independently review the blinded packet. Reconcile
  disagreements only after that pass. Preserve unresolved cases as uncertain;
  do not force labels to make a numeric target.
- Never tune prompts, heuristics, thresholds, or model selection on the final
  holdout.

The current 30-positive/30-benign set is a minimum smoke gate, not a broad
quality claim. Expand the benchmark when results show unsupported categories
or weak strata. Every claimed language/category needs enough independently
reviewed examples to report a meaningful per-stratum result; otherwise label
that area “insufficient evidence.”

### Run protocol

- Compare single-pass and staged red-team on exactly the same held-out cases,
  providers/models, and repetitions.
- Start only after independent adjudication is complete and recorded.
- Use at least three healthy model families and two repetitions to satisfy the
  current project gate. Additional repetitions are preferable for measuring
  run-to-run variability.
- Preflight provider availability and expected calls/cost. The current minimum
  matrix is approximately 2,160 calls before retries.
- Save a versioned manifest containing dataset hash, source revision, prompt
  and configuration hashes, model/provider identifiers, run settings,
  timestamps, retry/error status, latency, token usage, reported or estimated
  cost, and raw per-case outputs.
- Count provider errors, malformed responses, timeouts, refusals, and evidence
  gate failures as failed trials. Do not silently remove them from quality
  reporting.

### Report separate outcomes

Report raw counts and denominators, overall and per model family/category:

- finding precision and recall, false-positive burden, and missed findings;
- exact category, file, line, and evidence match;
- citation resolvability separately from semantic support of the claim;
- coverage, abstentions/uncertain cases, and provider completion/error rate;
- run-to-run consistency, latency, and cost where data are available;
- safe reproduction status for exploitability claims: reproduced, not
  reproduced, or not safely testable.

A matching quote establishes only that text appears at that location. It does
not establish that the text supports the claim, that an attacker can reach it,
or that the issue is exploitable. Model agreement and challenge-stage opinions
are not proof.

## Existing bounded gate

Keep the existing thresholds as project acceptance criteria, not as universal
standards:

- at least 30 independently reviewed positive and 30 benign held-out cases;
- at least three healthy, paired model families;
- at least two repetitions;
- pooled and per-family recall >= 85%, specificity >= 90%, and citation
  accuracy >= 95%;
- staged recall must not regress against single-pass;
- no hidden provider or evidence-gate failures.

A pass supports a bounded statement about the reviewed dataset, model roster,
and measured settings only. It does not prove completeness, exploitability, or
security of arbitrary projects.

## Work completed and remaining

### Completed

- Offline tests cover current fan-out, DAG ordering, dependency failure,
  checkpoint resume, and red-team evidence handling.
- Same-module request-hook presence no longer suppresses deterministic
  authorization candidates.
- The quality gate reports per-family metrics, rejects failed/unpaired and
  duplicate model rows, and cannot pass by silently dropping failed trials.
- Challenge opinions link to stable candidate IDs, include their reviewer
  model, and appear in final reports without overriding deterministic candidate
  status or claiming proof.
- Full offline test suite passes: **194 tests and 51 subtests**.
- Deterministic development smoke benchmark returns 12/12 positive findings,
  0 false positives across 12 clean cases, and exact source-line evidence on
  the 24-case development set. This is tuned development data, not independent
  quality evidence.
- Blinded reviewer packet is prepared; keep the answer key private until
  adjudication.

### Next actions

1. Obtain the independent second review of the 60 blinded cases.
2. Reconcile discrepancies and update labels only after the review is complete;
   retain reviewer notes and unresolved cases.
3. Confirm three healthy model families, call budget, and pricing without
   disclosing credentials.
4. Run paired single-pass and staged evaluations on the reviewed holdout.
5. Publish raw outcomes, per-family/per-category metrics, errors, uncertainty,
   limitations, and the exact bounded conclusion.
6. Add benchmark cases only for gaps demonstrated by this evaluation. External
   repositories are references and optional data sources, not runtime
   dependencies.

The live held-out evaluation remains intentionally blocked until the blinded
second review has been completed and labels reconciled. The user authorized
provider calls, but no provider evaluation has been run against provisional
labels.

## Use of external references

- OWASP BenchmarkJava is relevant only if Java review becomes an intended
  scope; it is a purpose-built scanner benchmark and GPL-2.0.
- PrimeVul is a possible C/C++ research dataset, but the dataset is separate
  from its code repository; verify data terms, provenance, and leakage risk
  before use.
- CodeQL query tests are a regression-harness pattern, not an LLM-quality
  benchmark. Review CodeQL's terms before using its CLI or bundle.
- Existing SmartBugs material is relevant only to Solidity coverage;
  intentionally vulnerable applications such as Juice Shop belong in isolated
  integration exercises, not ordinary CI or the primary labeled source set.
- Do not add another orchestration framework now. Study checkpoint, retry,
  timeout, replay, budget, and observability patterns only to fill measured
  gaps in OmniCLI's existing runner.
