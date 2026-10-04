# Ranked issues

**Filled by:** session 9 (the first list, `cap01-e5`), kept current until
session 14, which fixes rank 1 and adds its regression test.

At least three rows. Ranks 1, 2, 3... with no gap and no tie: two issues ranked
1 is a list nobody prioritised. The impact is what orders it.

The columns are the three fields `cap01-e5` reads.

| rank | issue | impact |
|---:|---|---|
| 1 | Lexical retrieval misses paraphrases with no word overlap | a user who asks in their own words gets a refusal for a supported question: "keeping a chatbot from being hijacked by hidden commands" is refused although prompt-injection.md answers it |
| 2 | When the model refuses or is not configured, the answer is the whole cited document, not the sentences that answer the question | the reader has to find the answer inside about 2,300 characters of quote, and the grader's phrase check cannot tell a precise answer from a long quote |
| 3 | An answer that needs two documents cites only one | a question whose answer spans two documents loses citation recall, and the reader is never pointed at the second source |

## Rank 1, in progress

- The fix: not written yet. The next step is query expansion measured on unseen paraphrases, not only on the cases it was written for (session 7 showed a synonym rule tuned on five cases dropped unseen probes from 75% to 50%).
- The regression test: `test_regression_rank_1_of_the_issue_list` in `tests/test_contract.py`, marked `xfail(strict=True)` until the fix lands.
- Before and after: see [EVAL_REPORT.md](EVAL_REPORT.md).
