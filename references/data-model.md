# Data model and invariants

## Purpose

The `english` schema separates content, evidence, memory state, practice history, errors, and proficiency. This avoids treating a vocabulary list as if it were a learning model.

## Entity groups

### Learner and policy

- `learners`: one learner profile, timezone, current range, configurable CEFR target, desired retention, and daily target.
- `study_policies`: versioned deterministic rules, intervals, scoring rubric, and mastery thresholds. Only one policy is active per learner.

Policy lives in data so future algorithm changes are auditable rather than silently changing historical meaning.

### Sources and reading evidence

- `sources`: books, articles, papers, podcasts, courses, and other materials.
- `reading_sessions`: one completed or open reading interaction, original and corrected summaries, dimension scores, feedback, and metrics.
- `encounters`: every observed encounter with a learning item, linked to source and session when known.
- `item_examples`: source or coach examples, with origin and quality.

An item and an encounter are different: `put off` is one item, but seeing it in three chapters creates three encounters. Repeated encounters enrich evidence without duplicating the item.

### Learning content

- `learning_items`: first-class words, phrases, phrasal verbs, collocations, idioms, grammar, morphology, pronunciation, and error patterns.
- `item_relations`: synonym, antonym, word-family, form, collocation, confusable, grammar, and other relationships.

Identity is learner + kind + normalized canonical form + sense key. Separate senses use separate rows. Multiword expressions remain intact. `forms` stores structured morphology such as irregular past and past participle.

### Memory and practice

- `memory_states`: one mutable current state per item: stage, due date, lapses, review counts, difficulty, and receptive/productive/contextual/grammar mastery.
- `review_events`: immutable audit log of every attempt, prompt, response, correction, score, confidence, and before/after memory state.
- `daily_plans` and `daily_plan_items`: persisted deterministic five-item selections. Once started, a plan is not reshuffled.
- `assessments` and `assessment_items`: persisted formal tests and item-level evidence.

Mutable state makes current selection efficient; immutable events preserve explainability and permit future re-estimation. Never edit history to make a score look better.

### Errors, insights, and proficiency

- `error_patterns`: deduplicated reusable weaknesses with occurrence count, severity, status, and optional linked study item.
- `error_events`: individual evidence instances linked to a reading session, review, assessment, or item.
- `insights`: stable-key strengths, weaknesses, trends, recommendations, milestones, and warnings. Recurring evidence upserts the same insight.
- `proficiency_snapshots`: sparse CEFR-oriented measurements with confidence, dimensions, metrics, and evidence.

An error event is raw evidence; an error pattern is a recurring target; an insight is a coaching conclusion; a proficiency snapshot is a higher-level estimate. Keep those layers separate.

### Check-in state

- `checkins`: records whether a local-date reading check-in is pending, started, completed, skipped, or missed. It is database state, not a scheduler. ChatGPT built-in scheduled activities are configured separately.

## Deterministic selection

`english.generate_daily_plan` persists at most the requested count using this strict order:

1. due or overdue reviewed items, earliest due first;
2. unreviewed items, oldest first-seen first;
3. weak or lapsed items, weakest productive/contextual evidence first;
4. oldest remaining items.

Stable tie-breakers end with item UUID. No randomness means the same database state yields the same plan.

`english.generate_assessment` similarly prioritizes open errors, due items, weak or lapsed items, and items least recently assessed.

## State transitions

Only `english.record_review` changes memory stage, due date, mastery, lapses, and status. Scores are 0-4. The active interval policy is `[1,3,7,14,30,60,120,240,365]` days, with failures returning sooner. The due timestamp is determined by the active database policy in the learner-configured timezone; inspect that policy rather than assuming a fixed time.

A normal lexical item reaches `mastered` only with sufficient stage, consecutive success, productive mastery, and contextual mastery. Grammar and pronunciation use dimension-appropriate thresholds. Recognition alone cannot produce a robust human-facing mastery claim.

## Write API

Use these functions rather than direct table writes:

- `english.ingest_item`
- `english.generate_daily_plan`
- `english.record_review`
- `english.save_reading_checkin`
- `english.log_error`
- `english.generate_assessment`
- `english.record_assessment_answer`
- `english.save_insight`
- `english.save_proficiency_snapshot`
- `english.ensure_checkin`
- `english.get_nightly_context`

See `database-contract.md` for exact arguments and examples.

## Security model

The schema is private and intended for the authenticated Supabase MCP project connection. RLS is enabled, no anon/authenticated policies exist, and public Data API roles have no schema, table, sequence, or function privileges. Do not add browser-facing grants during ordinary coaching.