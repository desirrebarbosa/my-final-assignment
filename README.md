# my-final-assignment

A research assistant that answers developer questions about agents, RAG, structured outputs, MCP, prompt injection and evaluation from six versioned documents, cites the one document it used, and refuses with a flag for human review when those documents say nothing.

<!-- add the CI badge once the repository exists:
![check](https://github.com/<your-github-username>/my-final-assignment/actions/workflows/check.yml/badge.svg) -->

## The problem

A developer asking a chat model about agent engineering gets a fluent answer with no way to check where it came from, and the same confident tone when the model is guessing. A wrong answer about validation or injection defenses ships into code. This assistant only answers from documents you can open, names the document, and says "I don't know" instead of improvising.

## Demo

Two runs, pasted exactly as the commands printed them. Never an edited one.
`trace` prints every step the agent took, then the answer.

### One supported answer

```bash
uv run bootcamp capstone trace "How does chunking work in RAG?"
```

```text
[retrieve] top_k=8 -> [('rag-basics', 1, 4.25), ('rag-basics', 2, 4.25), ('evaluation-basics', 0, 2.53), ('prompt-injection', 2, 2.53), ('prompt-injection', 1, 1.72), ('rag-basics', 0, 1.72), ('rag-basics', 3, 1.72)]; citing rag-basics
[llm_call] attempt 1: 121 chars
[decision] the model refused or cited [], but rag-basics answers it; answering with the excerpt

answer: From rag-basics: RAG Basics Retrieval-augmented generation (RAG) grounds a model's answer in documents you control. Instead of hoping the model memorized the right facts, the application retrieves relevant passages at question time and places them in the prompt. The model's job shrinks from "know everything" to "synthesize from what you were shown" — a job it is much better at. The pipeline A minimal RAG pipeline has four stages. Chunking splits documents into passages small enough to be individually relevant — respecting paragraph boundaries beats cutting at a fixed character count mid-sentence. Indexing prepares chunks for search; a lexical index (keyword overlap) is a perfectly respectable baseline and is deterministic, cheap, and debuggable. Embedding-based semantic search improves recall on paraphrased questions but adds a dependency and a failure mode you cannot inspect by eye. Retrieval selects the top-k chunks for a query. Generation answers using only the retrieved context, with citations naming the source documents. Retrieval failure vs generation failure When a RAG system answers wrongly, the first diagnostic question is: did the right passage reach the prompt? If retrieval missed it, no amount of prompt engineering will fix the answer — fix chunking, the index, or the query instead. If the right passage was present and the answer still went wrong, the failure is in generation: instructions, formatting, or the model ignoring context. Keeping retrieved chunks inspectable (log them, show them) is what makes this diagnosis a two-minute check instead of a guessing game. Citations keep the system honest Every answer should name the document ids it drew from, and the application should verify those ids against what was actually retrieved. A citation the retriever never returned is a fabrication and should be stripped and flagged. The "I don't know" case matters most: when nothing relevant is retrieved, the system must refuse rather than let the model improvise from its training data. Small corpus, high leverage Most of RAG's value appears with a corpus of five documents, not five million. Start small, keep the corpus versioned in git, and grow it only when a measured retrieval failure says you need to.
citations: ['rag-basics']
confidence: 0.6
needs_human_review: False
```

### One refusal

```bash
uv run bootcamp capstone trace "What is the capital city of Mongolia?"
```

```text
[retrieve] top_k=8 -> []
[decision] no document covers the question's terms; refusing without an LLM call

answer: I don't know based on the provided corpus.
citations: []
confidence: 0.0
needs_human_review: True
```

## Architecture

One run is a hand-written loop in `agent.py`:

1. **Retrieve** the top 8 passages. The cited document is the one that covers most of the question's content words, then the highest summed score.
2. **Refuse before any model call** when no document covers enough of the question (at least 60% of its content words, or 50% with a summed score of 9 or more). Cost: 0 model calls.
3. **Scan** the top passages for instruction-shaped text (an order written as its own sentence). A hit flags the answer for human review.
4. **Ask the model once**, bounded by `timeout_s`, with one corrective retry on invalid JSON. A provider that raises or hangs becomes a flagged refusal. Cost: 1 model call, 2 at most.
5. **Verify citations** against what was retrieved. A fabricated citation is stripped and the answer flagged.
6. **Quote the cited document** under the model's answer. If the model refuses although retrieval was strong, the quote is the answer.

See [docs/adr/0001-run-shape.md](docs/adr/0001-run-shape.md).

## Measured results

Every number here comes from a command in this table, run on this commit. Say
which model produced it: CI has no keys, so a CI number is always the offline
fake model's.

| What | Command | Model | Result |
|---|---|---|---|
| Contract tests | `uv run pytest` | fake | `7 passed, 1 skipped, 1 xfailed` |
| Practice grader | `uv run bootcamp capstone grade` | fake; ollama `qwen2.5:3b-instruct` | `score: 10/10 (100%) — pass bar 30% — PASSED` on both |
| Evaluation, before and after | see [docs/EVAL_REPORT.md](docs/EVAL_REPORT.md) | fake | `3/10 (30%)` before, `10/10 (100%)` after |

## The honest limitation

Lexical retrieval misses paraphrases with no word overlap: "keeping a chatbot from being hijacked by hidden commands" is refused although prompt-injection.md answers it. The next step is query expansion, measured on paraphrases it was not written for, with `test_regression_rank_1_of_the_issue_list` turning green as the proof.

The full ranked list is in [docs/ISSUES.md](docs/ISSUES.md).

## How to run it

```bash
git clone https://github.com/<your-github-username>/my-final-assignment && cd my-final-assignment && uv sync && uv run pytest
```

No key needed: without a `.env` it runs on the offline fake model. For a real
model, copy `.env.example` to `.env`, fill in your provider, and
`uv sync --extra anthropic` (or `--extra openai`).

To hand in the final assignment, commit and push, then run
`uv run bootcamp capstone submit --github <you>`. It runs the practice set
first, then answers the final questions and opens the pull request.
`--dry-run` shows the bundle without handing anything in.

## Credits

- [Gecko-Academy/dev3pack-cohort-2026-09](https://github.com/Gecko-Academy/dev3pack-cohort-2026-09): the course package this agent runs on (retrieval, the answer schema and parser, the tool registry, the refusal text), and the citation-normalising `_as_ids` adapted from `bootcamp_agent/agent.py`.

## Rollback

If a change drops the practice score below 10/10 or fails any critical question, `git revert` that commit within 1 day and re-run `uv run bootcamp final grade` before pushing again.

---

| Path | What it is |
|---|---|
| `agent.py` | The agent: `YourAgent`, the class the tests, `trace` and the grader run |
| `tests/test_contract.py` | The capstone contract, as tests (`uv run pytest -k refusal`, `-k injection`, ...) |
| `data/corpus/` | The six source documents, versioned; nothing here writes to them |
| `docs/EVAL_REPORT.md` | Numbers you produced, before and after, with the command behind each |
| `docs/SKILL.md` | A skill another assistant can load (session 10) |
| `docs/adr/0001-run-shape.md` | The architecture decision and what would reverse it (session 10) |
| `docs/RETENTION.md` | What a session remembers, and what it refuses to (session 11) |
| `docs/ISSUES.md` | The ranked issue list (session 9, kept until 14) |

Built during the Dev3Pack AI Engineering bootcamp, on the course package at
commit `1de8649529156c35bdeafacfbbef40726a9b0fba` of https://github.com/Gecko-Academy/dev3pack-cohort-2026-09.
