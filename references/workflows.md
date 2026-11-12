# Complete workflows

## Workflow 1: nightly reading check-in

### Trigger

Use when a ChatGPT scheduled activity invokes the optional scheduled check-in or the learner requests it manually.

### Step 1: initialize from the database

1. Determine the current date in the privately configured learner IANA timezone.
2. Call `english.ensure_checkin` for that date.
3. Call `english.get_nightly_context`.
4. Read the latest active error patterns, due-item count, recent scores, and last source from the returned context.
5. Do not announce database internals unless relevant.

### Step 2: opening prompt

Ask one compact question in Spanish, requesting English production:

> ¿Qué leíste hoy? Dime el título o la fuente y escribe en inglés entre 3 y 6 oraciones sobre lo que pasó o lo que aprendiste. Al final, agrega las palabras, frases o phrasal verbs que marcaste y, si puedes, la oración donde aparecieron.

If the scheduled activity prompt already supplied the source or requested a different format, preserve that information and ask only for what is missing.

### Step 3: interpret the response

Extract:

- source title, type, and author when stated;
- original English summary;
- reading duration if stated;
- marked items and their source sentences;
- learner guesses and confidence if stated;
- a difficult passage or unresolved question;
- candidate error patterns.

Do not require metadata the learner did not track. Use `other` and a neutral source title such as `Unspecified reading` only when necessary to save a valid session.

### Step 4: correct and evaluate

1. Produce a corrected version that preserves the learner's intended meaning and approximate voice.
2. Explain only the 1-3 most valuable corrections.
3. Distinguish:
   - grammar error,
   - lexical choice or collocation,
   - register or naturalness,
   - mechanics,
   - possible comprehension problem.
4. Score all six dimensions using the fixed rubric.
5. Calculate the weighted overall score exactly.
6. Treat uncertain interpretation as uncertainty, not as a comprehension error.

### Step 5: persist the session

Call `english.save_reading_checkin` and capture the returned reading session UUID. Store:

- raw learner report;
- original and corrected summaries;
- focused coach feedback;
- all dimension scores;
- metrics such as summary word count, number of reported items, and substantive errors.

### Step 6: persist learning items

For each worthwhile item:

1. Classify and normalize it.
2. Preserve the source sentence.
3. Store form, meaning, use, and relevant morphology.
4. Call `english.ingest_item` with the reading session UUID.
5. If the same expression has two clearly different senses in the response, create two sense keys.

Store grammar topics such as past participles, article use, or present-perfect contrast when they are a recurring learning target, not merely because one sentence contains that structure.

### Step 7: persist errors

For each substantive reusable error:

1. Create a stable pattern key.
2. Assign category, subtype, and severity.
3. Call `english.log_error` linked to the reading session.
4. Set `p_should_study => true` when the error is repeated, severity 2-3, or central to the learner's configured next CEFR target.

Do not log an obvious accidental typo unless it recurs.

### Step 8: persist durable insights

Save only findings that will improve later coaching. Use `english.save_insight` with one to three stable keys:

- session-specific: `reading:<YYYY-MM-DD>:<dimension>`;
- recurring weakness or strength: `trend:<category>:<subtype>`;
- current recommendation: `recommendation:<training-target>`.

Include measurable evidence such as scores, error counts, item UUIDs, or review modes. Do not store generic praise or a transient mood.

### Step 9: close the check-in

Use a compact structure:

- corrected version;
- scores in one line;
- saved items;
- one priority for the next session.

Do not automatically run the five-item review. The scheduled activity for the check-in and the future evaluation activity are separate.

## Workflow 2: evaluation

### Choose the evaluation mode

- `review`, `practice`, `study five`, or no explicit exam language: use the five-item review.
- `exam`, `test`, `diagnostic`, `progress test`, or a scheduled evaluation activity: use formal assessment.

When ambiguous, use the five-item review because it is shorter and formative.

## Workflow 2A: five-item review

### Step 1: fetch the plan

Call `english.generate_daily_plan` for the local date, count 5, and no regeneration.

Keep:

- plan UUID;
- item UUID;
- ordinal;
- kind;
- canonical form;
- meaning and definition;
- memory status;
- stage;
- due date;
- selection reason.

### Step 2: run one item

1. Choose the deterministic mode from the kind and stage.
2. Build a prompt that tests one sense.
3. Hide the answer.
4. Ask for the answer and optional confidence code.
5. Do not move to the next item in the same message.

### Step 3: grade and persist

1. Grade 0-4.
2. Write a corrected answer.
3. Give one focused explanation.
4. Call `english.record_review` with `origin_type => 'daily_plan'` and the plan UUID.
5. Use returned scheduling values in any displayed next-review statement.
6. If there is a reusable error, call `english.log_error` linked to the review event.

### Step 4: finish

After all plan items:

- report total points out of `4 x item_count`;
- list items scoring below 3;
- mention high-confidence errors separately;
- give one short practice recommendation derived from the weakest mode or error pattern.

## Workflow 2B: formal assessment

### Step 1: generate

Map the request to a purpose:

- initial broad level check: `diagnostic`
- delayed-memory check: `retention`
- change over time: `progress`
- grammar-specific: `grammar`
- vocabulary-specific: `vocabulary`
- broad exam: `mixed`

Call `english.generate_assessment`. Default to 10 items.

### Step 2: administer

For each returned item:

1. Use the suggested mode unless it is impossible in the current modality.
2. Ask one prompt and do not reveal the answer.
3. Do not teach before grading.
4. Do not give hints in formal mode. If one is requested, provide the smallest useful cue and cap the score at 1.
5. Record every answer immediately.

### Step 3: grade

Use the 0-4 rubric. At B2-C1, distinguish:

- merely understandable from natural;
- correct denotation from correct collocation;
- grammatical form from appropriate register;
- memorized definition from flexible use.

### Step 4: persist

Call `english.record_assessment_answer` after every answer. Log reusable errors linked to the assessment item.

After the final item, query the assessment and all item rows. If the assessment function has not marked it complete, do not invent a completed state; report the stored item results and state what remains.

### Step 5: persist conclusions and report

1. Upsert the strongest durable finding with `english.save_insight`, using an assessment-scoped or longitudinal stable key.
2. If the evidence threshold is met, call `english.save_proficiency_snapshot`; otherwise explicitly avoid creating a level snapshot.
3. Report:
   - total score and percentage;
   - performance by tested dimension or mode;
   - strongest evidence;
   - the top two error patterns;
   - one priority for the next two weeks.

Phrase level estimates as evidence-based ranges with confidence.

## Workflow 3: passive capture during another conversation

The learner may mention terms while discussing a book without asking for a formal check-in. When the intent to remember them is clear:

1. Explain the term in context.
2. Store it immediately through `english.ingest_item`.
3. Confirm the saved canonical form and sense.
4. Continue the original conversation; do not force a quiz.

If intent is unclear and storage would materially change the user's persistent learning record, ask one concise question: `Lo guardo para tus repasos?`