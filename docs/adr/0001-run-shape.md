# ADR 0001: the shape of one run

**Filled by:** session 10, for the choice you measured in session 8 (chain,
loop or graph, and the model calls each one cost). The four fields are the
ones `ch10-e2` reads.

- Status: accepted
- Date: 2026-10-04

## Context

The agent needs retrieval, at most one model call, citation checks and three refusal paths. In session 8 the plain loop spent 1 model call per question, the langgraph capstone graph 2 (answer, critique) and the four-role subagents graph 3 (router, answerer, critic), while its router stopped an out-of-corpus question after 1.

## Decision (`decision`)

We keep the hand-written loop in agent.py, with one model call per question, instead of moving the assistant to a langgraph graph.

## Options considered (`options_considered`)

1. The hand-written loop in agent.py
2. A langgraph graph with declared nodes and edges

## Why not the other option (`why_not`)

The graph adds a dependency and spent 3 model calls where the loop spends 1, and the refusals a router would buy are already made before any model call by the retrieval coverage check.

## What would reverse it (`reverses_it`)

We switch to the graph if the workflow grows past 8 states, or if a retry bug takes more than 2 hours to trace through the loop.
