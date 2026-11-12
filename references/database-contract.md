# Supabase database contract

## Fixed target

- Project ID: `<SUPABASE_PROJECT_ID>`
- Schema: `english`
- Learner slug: configured privately; `example_learner` below is a fictional SQL example only.
- Timezone: configured privately; use the learner's IANA timezone.

All examples are SQL passed to Supabase MCP `execute_sql` with the **privately configured project ID**. The example slug `example_learner`, dates, content, and UUIDs are fictional. Replace the slug with the configured value *as a SQL string value*, not as an identifier; never execute the examples verbatim on real data.

## Safety for user text

Never accept SQL fragments from the user. When embedding a text value in SQL:

1. Replace each single quote `'` with two single quotes `''`.
2. Put the result inside a normal PostgreSQL string literal.
3. Use only fixed function and parameter names from this file.
4. Pass missing values as `null`, never as the text `'null'`.
5. For JSON, construct valid JSON in the tool call and cast it to `jsonb`, or use `jsonb_build_object` with safely escaped scalar values.
6. Cast every `smallint` argument explicitly, for example `5::smallint`; PostgreSQL does not always resolve named calls from an `integer` literal to a `smallint` parameter.

## Read learner and policy

```sql
select
  l.slug, l.timezone, l.current_cefr_low, l.current_cefr_high,
  l.target_cefr, l.daily_item_target, l.desired_retention,
  p.algorithm_version, p.intervals_days, p.score_rubric,
  p.selection_policy
from english.learners l
join english.study_policies p
  on p.learner_id = l.id and p.active
where l.slug = 'example_learner';
```

## Capture an item

Use named parameters. Omit nothing from the function contract; pass `null` or empty JSON/arrays where data is unavailable.

```sql
select english.ingest_item(
  p_learner_slug       => 'example_learner',
  p_kind               => 'phrasal_verb',
  p_canonical_form     => 'put off',
  p_sense_key          => 'verb:postpone',
  p_definition_en      => 'to delay doing something',
  p_meaning_es         => 'posponer; aplazar',
  p_part_of_speech     => 'phrasal verb',
  p_phonetic           => null,
  p_register           => 'neutral',
  p_cefr_estimate      => 'B1',
  p_cefr_confidence    => 0.80,
  p_frequency_band     => 'high',
  p_word_family_root   => 'put',
  p_grammar_category   => null,
  p_forms              => '{"past":"put off","past_participle":"put off","ing":"putting off"}'::jsonb,
  p_tags               => array['reading','multiword'],
  p_notes              => null,
  p_metadata           => '{}'::jsonb,
  p_reading_session_id => null,
  p_source_id          => null,
  p_context_sentence   => 'They put off the meeting until Friday.',
  p_user_guess         => null,
  p_known_before       => false,
  p_confidence         => null,
  p_lookup_used        => true,
  p_encountered_at     => now()
) as item_id;
```

Valid `p_kind` values:

`word`, `phrase`, `phrasal_verb`, `collocation`, `idiom`, `grammar`, `morphology`, `pronunciation`, `error_pattern`.

Identity is learner + kind + normalized canonical form + normalized sense key. Re-ingestion records a new encounter and updates the last-seen state without creating an accidental duplicate.

## Five-item deterministic plan

```sql
select *
from english.generate_daily_plan(
  p_learner_slug => 'example_learner',
  p_plan_date => date '2025-01-15',
  p_target_count => 5::smallint,
  p_regenerate => false
);
```

Never set `p_regenerate => true` after the learner has answered any item in that plan.

Selection tiers, in order:

1. reviewed items that are due or overdue, earliest due first;
2. unreviewed items, oldest first-seen first;
3. weak or lapsed items, weakest productive/contextual mastery first;
4. oldest remaining items.

Tie breakers are deterministic and end with item UUID. No randomness is allowed.

## Record one review

Use the item UUID returned by the plan. `origin_id` should be the daily plan UUID when the review came from a daily plan.

```sql
select *
from english.record_review(
  p_learner_slug      => 'example_learner',
  p_item_id           => '00000000-0000-0000-0000-000000000000'::uuid,
  p_mode              => 'cloze',
  p_score             => 3::smallint,
  p_response          => 'The learner response',
  p_expected_answer   => 'The expected answer',
  p_corrected_answer  => 'The corrected answer',
  p_prompt             => 'The exact prompt shown',
  p_confidence         => 4::smallint,
  p_response_time_ms  => null,
  p_feedback           => 'Brief Spanish feedback',
  p_origin_type        => 'daily_plan',
  p_origin_id          => '00000000-0000-0000-0000-000000000000'::uuid,
  p_evaluator          => 'chatgpt',
  p_metadata           => '{"hint_used":false}'::jsonb,
  p_reviewed_at        => now()
);
```

Valid review modes:

`recognition`, `definition_recall`, `en_to_es`, `es_to_en`, `production`, `cloze`, `word_form`, `context_sentence`, `paraphrase`, `collocation`, `phrasal_verb`, `grammar_transform`, `error_correction`, `other`.

The function returns the new status, stage, due date, and mastery dimensions. Do not modify these values separately.

## Nightly context and check-in row

```sql
select english.ensure_checkin(
  p_learner_slug => 'example_learner',
  p_local_date => date '2025-01-15'
) as checkin_id;

select english.get_nightly_context(
  p_learner_slug => 'example_learner',
  p_local_date => date '2025-01-15'
) as context;
```

The context includes dashboard counts, the last source, recent reading scores, open error patterns, active insights, and any persisted daily plan. It creates a database check-in row only; it does not schedule a ChatGPT activity.

## Save a reading check-in

Call this after the learner has answered and the coach has produced the corrected version and scores.

```sql
select english.save_reading_checkin(
  p_learner_slug        => 'example_learner',
  p_session_date        => date '2025-01-15',
  p_source_title        => 'Book title',
  p_source_type         => 'book',
  p_source_author       => null,
  p_raw_report          => 'Full learner report',
  p_summary_original    => 'Original English summary',
  p_summary_corrected   => 'Corrected English summary',
  p_difficult_passage   => null,
  p_coach_feedback      => 'Focused Spanish feedback',
  p_minutes_read        => 60,
  p_words_read_estimate => null,
  p_scores              => '{
    "comprehension": 82,
    "grammar": 74,
    "vocabulary": 78,
    "coherence": 80,
    "mechanics": 85,
    "overall": 79
  }'::jsonb,
  p_metrics             => '{
    "new_items_reported": 4,
    "substantive_errors": 2,
    "summary_word_count": 93
  }'::jsonb
) as reading_session_id;
```

Valid source types:

`book`, `article`, `essay`, `news`, `paper`, `video`, `podcast`, `conversation`, `course`, `other`.

After this returns, pass the resulting reading session UUID into every related `ingest_item` and `log_error` call.

## Log an error

Use one stable key per reusable pattern. Recommended format:

`<category>:<subtype>:<normalized-target>`

Examples:

- `tense:past-simple:finished-time`
- `verb-form:past-participle:write-written`
- `collocation:verb-noun:make-a-decision`
- `article:missing:singular-count-noun`
- `preposition:wrong:interested-in`

```sql
select english.log_error(
  p_learner_slug       => 'example_learner',
  p_pattern_key        => 'verb-form:past-participle:write-written',
  p_category           => 'verb_form',
  p_subtype            => 'past_participle',
  p_description_es     => 'Confunde el pasado simple con el participio pasado.',
  p_original_text      => 'I have wrote three pages.',
  p_corrected_text     => 'I have written three pages.',
  p_explanation_es     => 'Después de have se usa el participio: written.',
  p_severity           => 2::smallint,
  p_reading_session_id => '00000000-0000-0000-0000-000000000000'::uuid,
  p_review_event_id    => null,
  p_assessment_item_id => null,
  p_item_id            => null,
  p_should_study       => true,
  p_metadata           => '{}'::jsonb
) as pattern_id;
```

Severity:

- 1: local mistake that does not obstruct meaning;
- 2: meaningful grammar, form, collocation, or lexical precision problem;
- 3: repeated or high-impact error that changes or blocks meaning.

## Generate a formal assessment

```sql
select *
from english.generate_assessment(
  p_learner_slug   => 'example_learner',
  p_purpose        => 'mixed',
  p_item_count     => 10::smallint,
  p_assessment_date => date '2025-01-15',
  p_regenerate     => false
);
```

The function returns assessment and item UUIDs, order, learning item, kind, canonical form, suggested mode, and selection reason. Use the returned order exactly.

## Record one assessment answer

```sql
select english.record_assessment_answer(
  p_learner_slug       => 'example_learner',
  p_assessment_item_id => '00000000-0000-0000-0000-000000000000'::uuid,
  p_score              => 3::smallint,
  p_response           => 'The learner response',
  p_corrected_answer   => 'The corrected answer',
  p_prompt             => 'The exact prompt',
  p_expected_answer    => 'Expected answer or scoring criteria',
  p_confidence         => 4::smallint,
  p_response_time_ms   => null,
  p_feedback           => 'Brief Spanish feedback',
  p_metadata           => '{"hint_used":false}'::jsonb
) as review_event_id;
```

After the last answer, read the assessment and all assessment items:

```sql
select
  a.id, a.assessment_date, a.purpose, a.status,
  a.total_score, a.max_score, a.dimension_scores, a.summary,
  ai.ordinal, ai.mode, ai.prompt, ai.response, ai.corrected_answer,
  ai.score, ai.max_score, ai.confidence, ai.feedback,
  li.kind, li.canonical_form
from english.assessments a
join english.assessment_items ai on ai.assessment_id = a.id
left join english.learning_items li on li.id = ai.item_id
where a.id = '00000000-0000-0000-0000-000000000000'::uuid
order by ai.ordinal;
```

## Save or update a durable insight

Use a stable key. The function upserts by learner plus `insight_key`, so recurring evidence updates the same longitudinal finding.

```sql
select english.save_insight(
  p_learner_slug       => 'example_learner'::text,
  p_insight_key        => 'trend:grammar:past-participle'::text,
  p_insight_type       => 'weakness'::text,
  p_title              => 'Past participles remain fragile'::text,
  p_content            => 'Two high-confidence errors appeared in productive use.'::text,
  p_evidence           => '{"error_count":2,"confidence_band":"high"}'::jsonb,
  p_recommended_action => 'Use a verb-form transformation task in the next review.'::text,
  p_priority           => 1::smallint,
  p_period_start       => date '2025-01-15',
  p_period_end         => date '2025-01-15',
  p_status             => 'active'::text
) as insight_id;
```

Valid insight types: `strength`, `weakness`, `trend`, `recommendation`, `milestone`, `warning`. Priority `1` is highest. Do not store generic praise.

## Save a proficiency snapshot

Call this only when the evidence threshold in `pedagogy-and-rubrics.md` is met. Scores use 0-100 and confidence uses 0-1.

```sql
select english.save_proficiency_snapshot(
  p_learner_slug                 => 'example_learner'::text,
  p_basis                        => 'assessment'::text,
  p_overall_cefr_estimate        => 'B2'::text,
  p_confidence                   => 0.78::numeric,
  p_reading_score                => 84::numeric,
  p_writing_score                => 76::numeric,
  p_grammar_score                => 72::numeric,
  p_vocabulary_receptive_score   => 86::numeric,
  p_vocabulary_productive_score  => 70::numeric,
  p_collocation_score            => 66::numeric,
  p_comprehension_score          => 84::numeric,
  p_error_rate_per_100_words     => 4.30::numeric,
  p_metrics                      => '{"review_items":24,"reading_checkins":3}'::jsonb,
  p_evidence                     => '{"window_days":14}'::jsonb,
  p_notes                        => 'Reading is stronger than productive lexical precision.'::text,
  p_measured_at                  => now()
) as snapshot_id;
```

Valid bases: `reading_checkin`, `assessment`, `weekly_rollup`, `manual`, `baseline`.

## Progress queries

Inventory and memory state:

```sql
select
  li.kind,
  count(*) as item_count,
  count(*) filter (where ms.status = 'mastered') as mastered,
  count(*) filter (where ms.due_at <= now() and ms.status <> 'suspended') as due_now,
  round(avg(ms.receptive_mastery) * 100, 1) as receptive_pct,
  round(avg(ms.productive_mastery) * 100, 1) as productive_pct,
  round(avg(ms.contextual_mastery) * 100, 1) as contextual_pct,
  round(avg(ms.grammar_mastery) * 100, 1) as grammar_pct
from english.learning_items li
join english.memory_states ms on ms.item_id = li.id
join english.learners l on l.id = li.learner_id
where l.slug = 'example_learner' and li.active
group by li.kind
order by li.kind;
```

Recent review performance by mode:

```sql
select
  mode,
  count(*) as attempts,
  round(avg(score)::numeric, 2) as avg_score,
  round(100.0 * avg((score >= 3)::int), 1) as success_pct
from english.review_events re
join english.learners l on l.id = re.learner_id
where l.slug = 'example_learner'
  and re.reviewed_at >= now() - interval '30 days'
group by mode
order by attempts desc, mode;
```

Open error patterns:

```sql
select
  pattern_key, category, subtype, description_es,
  occurrences, severity_max, status, first_seen_at, last_seen_at
from english.error_patterns ep
join english.learners l on l.id = ep.learner_id
where l.slug = 'example_learner'
  and ep.status in ('open','improving')
order by severity_max desc, occurrences desc, last_seen_at desc
limit 20;
```

Recent check-in scores:

```sql
select
  session_date, comprehension_score, grammar_score, vocabulary_score,
  coherence_score, mechanics_score, overall_score, coach_feedback
from english.reading_sessions rs
join english.learners l on l.id = rs.learner_id
where l.slug = 'example_learner'
  and rs.status = 'completed'
order by session_date desc
limit 14;
```

Latest snapshots:

```sql
select *
from english.proficiency_snapshots ps
join english.learners l on l.id = ps.learner_id
where l.slug = 'example_learner'
order by ps.measured_at desc
limit 5;
```

## Private access model

The `english` schema is private by default:

- RLS is enabled on every table.
- No anon or authenticated policies are present.
- Public Data API roles have no schema, table, sequence, or function privileges.
- The connected Supabase MCP project session performs administrative coaching operations.

Do not add public policies or grants during normal use.