# English Learning Brain

A reusable, privacy-safe ChatGPT skill for English reading, vocabulary capture, spaced retrieval practice, writing corrections, formal assessments, and evidence-based CEFR progress.

## What is included

- `SKILL.md`: coaching orchestration and guardrails
- `references/`: Supabase function contracts, data-model notes, scoring rules, workflows, and optional scheduling prompt templates
- `agents/openai.yaml` and `assets/icon.svg`: ChatGPT skill UI metadata
- `config.example.md`: **placeholder-only** configuration guide

## Setup

1. Import the `english-learning-brain/` directory as a ChatGPT skill.
2. Connect and authorize your own Supabase project via the Supabase connector. It must already have the `english` schema and functions described in `references/database-contract.md`. This repository documents the contract; **it does not provision the schema or functions**.
3. Provide your project ID, learner slug and IANA timezone **privately at runtime**, or in `config.local.md` that is ignored by Git. Do **not** commit or publish that file.
4. Read or configure the learner's CEFR band, goals and study policy in the private database. Run a review only after the connection is configured.

## Privacy and security

This public version intentionally excludes any live project ID/name, learner identifier, personal level, actual learner history, credentials and tokens. All values in SQL examples are fictional. No personal learning records or exports are included.

> Publishing only the public skill does not make a Supabase project public. Do not add public database grants, API tokens, or personally identifying learner content to the repository.

## Constraints

The skill depends on an authorized Supabase MCP connection and database-side routines. It does not run by itself without those dependencies. Scheduling is configured separately in ChatGPT and is not created by the skill.