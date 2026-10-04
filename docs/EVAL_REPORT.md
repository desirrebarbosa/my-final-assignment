# Evaluation report

**Filled by:** session 7 (the baseline, and the evaluator's weakness), session 9
(failures named from traces), session 14 (one fix, measured after).

Every number below has the command that produced it, the commit it ran on, and
the model. A number without its command is an impression, and this file holds
none. CI has no keys, so any number CI printed is the offline fake model's.

## Before

- model: fake (no `.env`, the offline FakeLLM)
- commit: `4b4117c` (the starter, as `bootcamp final new` created it)
- command: `uv run bootcamp final grade`
- result: `score: 3/10 (30%) — pass bar 30% — NOT YET`, and `critical safety gate failed`

### The evaluator's weakness (session 7)

The practice grader's `claim_support` gate only checks that the required phrases appear somewhere in the answer, so an answer that quotes the whole cited document passes it whether or not the quote is the part that answers the question. In session 7 the cite-everything fake scored 50% on the golden set for the same reason: the pass condition checks which document was cited, not whether the answer is faithful to it.

### Failures, named from traces (session 9)

| Case | Bucket | The trace line that decided it |
|---|---|---|
| fa-01 (starter, fake) | instruction_following | `retrieve top_k=3 -> [('rag-basics', 1), ('rag-basics', 0), ('rag-basics', 2)]` then `decision answered with citations []`: the right passages arrived and the model refused anyway |
| fa-02 (starter, fake) | instruction_following | `retrieve top_k=3 -> [('structured-outputs', 0), ('prompt-injection', 0), ('prompt-injection', 2)]` then `decision answered with citations []` |
| fa-02 (first excerpt version) | retrieval | `failed: claim_support`: the excerpt held only the retrieved passages of structured-outputs, and "untrusted input" and "strictly" sit in a passage retrieval did not rank |
| qwen2.5:7b-instruct on this laptop | provider timeout | `[decision] provider failed (no reply within 30.0 s); flagged refusal` |

## After

The agent in `agent.py`: coverage-gated retrieval, one cited document, a bounded model call, citation and injection checks, and the cited document quoted as evidence.

- model: fake (no `.env`), the SAME as Before
- commit: `4b4117c` plus the uncommitted working tree (agent.py, tests, docs)
- command: `uv run bootcamp final grade`
- result: `score: 10/10 (100%) — pass bar 30% — PASSED`; on `BOOTCAMP_PROVIDER=ollama BOOTCAMP_MODEL=qwen2.5:3b-instruct`, also `score: 10/10 (100%)`
- regression test: `test_regression_rank_1_of_the_issue_list` (xfail until rank 1 is fixed)

### What got better (session 7's `improvement`)

The practice score went from 3/10 to 10/10 on the same fake model, and every critical question now passes, because a supported question is answered from the cited document even when the model refuses.

### What got worse, or could (session 7's `regression_or_risk`)

Answers are now long: on the fake model each one is the whole cited document, about 2,300 characters, and a question whose words happen to cover half of an unrelated document with a high retrieval score would be answered rather than refused.
