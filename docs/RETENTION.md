# Retention policy

**Filled by:** session 11. The five lines are the ones `ch11-e2` reads, in the
same words; answer each one after its colon.

STORED: nothing between questions; each call to YourAgent reads the six corpus documents and the question, and keeps no history.

WHY: every answer must be checkable against the corpus alone, so no earlier question or user detail may shape it.

CORRECTED BY: there is nothing stored to correct; a wrong answer is fixed in agent.py or the corpus, and the next call uses the fix.

EXPIRES: immediately: the trace and the answer live only for the one call (0 seconds after it returns), with a cap of 0 stored episodes.

WE REFUSE TO REMEMBER: questions, answers, names, email addresses, API keys, tokens and anything a document asks us to keep.

## How the code enforces it

`YourAgent` has no attribute that outlives a call except the corpus and the tools. `test_memory_is_capped_reset_and_kept_per_user` stays skipped until the agent remembers something.
