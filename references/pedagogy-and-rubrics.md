# Pedagogy and rubrics

## Design principles

1. **Retrieval before review.** Ask for an answer before showing it. Retrieval practice improves delayed retention more than repeated exposure alone.
2. **Spacing over cramming.** Review items across expanding intervals and bring failed items back sooner. The database owns interval transitions.
3. **Feedback after an attempt.** Give the correct form and a focused explanation immediately after grading.
4. **Receptive plus productive control.** Recognition is useful but insufficient. Move toward recall, cloze, collocation, transformation, paraphrase, and original production.
5. **Form, meaning, and use.** A learned item includes spelling or sound, meaning or sense, grammar, inflection, collocation, register, and contextual use where relevant.
6. **Context plus deliberate practice.** Capture items from real reading, then revisit them deliberately. Incidental encounters and intentional study are complementary.
7. **Multiword units are first-class.** Treat phrasal verbs, collocations, idioms, discourse markers, and fixed phrases as items, not annotations on a single word.
8. **Precision for C1.** Progress from broad meaning to natural collocation, register, nuance, paraphrase, and flexible production.
9. **Errors are evidence.** Track reusable patterns, not every typo. Repeated errors become study targets.
10. **Calibration matters.** Record confidence to detect false fluency and low-confidence correct knowledge.

## Evidence base

The workflow is based on the following established sources:

- Council of Europe, *CEFR Companion Volume* (2020): action-oriented proficiency descriptors, lexical range, control, precision, coherence, and mediation. Source: https://www.coe.int/en/web/common-european-framework-reference-languages/cefr-companion-volume-and-its-language-versions
- Roediger and Karpicke (2006), *Test-Enhanced Learning*: delayed retention benefits from retrieval practice. DOI: https://doi.org/10.1111/j.1467-9280.2006.01693.x
- Cepeda et al. (2006), meta-analysis of distributed practice: spacing improves verbal recall and optimal lag depends on the retention horizon. DOI: https://doi.org/10.1037/0033-2909.132.3.354
- Karpicke and Roediger (2010), *Is expanding retrieval a superior method for learning text materials?*: repeated retrieval and feedback improve retention, while expanding spacing is not consistently superior to equal spacing. DOI: https://doi.org/10.3758/MC.38.1.116
- Webb (2007), *The Effects of Repetition on Vocabulary Knowledge*: repeated encounters develop multiple aspects of word knowledge. DOI: https://doi.org/10.1093/applin/aml048
- Nation (2013), *Learning Vocabulary in Another Language*, second edition: knowing a word involves multiple dimensions; context learning and deliberate learning are complementary. DOI: https://doi.org/10.1017/CBO9781139858656

The exact interval schedule in this system is a practical policy, not a claim that one universal interval sequence is optimal for every learner.

## Review score rubric: 0-4

Use this rubric for every review and assessment item.

### 0 - no usable recall

- Blank, `I do not know`, unrelated answer, wrong sense, or form that fails the task.
- A response that changes the target meaning substantially.
- Schedule as a lapse through the database function.

### 1 - partial with decisive help or major error

- Some relevant knowledge, but a major meaning, form, particle, tense, or construction error remains.
- Correct only after a strong hint or answer-revealing prompt.
- Any hint requested before the answer caps the score at 1.

### 2 - essential idea correct, control incomplete

- Core meaning is correct, but there is a meaningful error in inflection, syntax, collocation, preposition, register, spelling, or precision.
- Understandable but not yet reliable for independent production.

### 3 - correct and independent

- Correct without help.
- At most a minor imprecision that does not alter meaning or natural use.
- For productive tasks, the sentence is grammatical and the target item fits the context.

### 4 - precise, natural, and flexible

- Correct, independent, idiomatic, and appropriately registered.
- Demonstrates nuance, a strong collocation, an accurate paraphrase, or a natural original context.
- Do not award 4 for bare recognition alone unless the task was unusually discriminating.

## Confidence scale: C0-C5

- C0: pure guess
- C1: very unsure
- C2: somewhat unsure
- C3: moderately sure
- C4: sure
- C5: completely sure

Use confidence diagnostically. A wrong C4-C5 answer indicates a misconception and should be highlighted. A correct C0-C2 answer indicates fragile knowledge.

## Deterministic prompt-mode selection

Use the memory stage returned by `generate_daily_plan`. Do not randomize.

### Words and general phrases

- stage 0: `definition_recall` from a simple English definition or context cue
- stage 1: `es_to_en`
- stage 2: `cloze`
- stage 3: `word_form` when morphology is relevant; otherwise `context_sentence`
- stage 4: `production`
- stage 5: `paraphrase`
- stage 6 or higher: cycle by `stage mod 4`
  - 0: `cloze`
  - 1: `production`
  - 2: `paraphrase`
  - 3: `context_sentence`

### Phrasal verbs

- stage 0: `definition_recall`
- stage 1: `cloze` with the particle hidden
- stage 2: `phrasal_verb` transformation from a single-word synonym
- stage 3 or higher: `context_sentence`, then alternate with `paraphrase`

### Collocations

- stage 0: `definition_recall`
- stage 1: `collocation` completion
- stage 2: `cloze`
- stage 3 or higher: `production`, then alternate with `error_correction`

### Idioms

- stage 0: infer meaning from context
- stage 1: `definition_recall`
- stage 2: `cloze`
- stage 3 or higher: `production` with register and pragmatic appropriateness

### Grammar

- stage 0: choose or explain the rule in a minimal contrast
- stage 1: `grammar_transform`
- stage 2: `error_correction`
- stage 3 or higher: constrained production, then a short free-production task

### Morphology and verb forms

- stage 0: `word_form`
- stage 1: `grammar_transform`
- stage 2: `cloze`
- stage 3 or higher: `production`

### Pronunciation

In text-only mode, test stress, syllable division, minimal-pair discrimination, or phonetic recognition. If voice input is available, ask the learner to pronounce it, but do not pretend to assess audio that was not received.

### Error patterns

- stage 0: `error_correction`
- stage 1: explain the correction
- stage 2: `grammar_transform`
- stage 3 or higher: original production designed to elicit the corrected pattern

## Prompt construction rules

- Keep the target hidden unless the mode requires showing it.
- Use one unambiguous sense.
- Prefer the learner's source context but alter enough wording to require retrieval rather than visual memory.
- For C1-oriented practice, test collocation, register, nuance, and paraphrase rather than obscure dictionary trivia.
- Avoid multiple-choice unless testing subtle discrimination. Free recall is the default.
- Accept valid regional variants and natural synonyms when they satisfy the task.

## Nightly check-in scoring: 0-100

Score from the learner's English summary and discussion, not from the difficulty of the source alone.

### Comprehension

- 90-100: accurate main ideas, important details, inference, and relationships
- 75-89: main ideas accurate with small omissions or one minor misunderstanding
- 60-74: partial understanding; important omissions or weak causal links
- 40-59: fragmented or several substantive misunderstandings
- 0-39: little evidence of understanding

### Grammar

- 90-100: high control of complex forms; rare minor errors
- 75-89: generally controlled; errors do not obscure meaning
- 60-74: recurring tense, agreement, article, preposition, or clause errors
- 40-59: frequent errors that sometimes obscure meaning
- 0-39: control too limited to communicate the summary reliably

### Vocabulary

- 90-100: precise, varied, natural collocation and register
- 75-89: adequate range with occasional repetition or imprecision
- 60-74: understandable but limited, generic, or translated-sounding
- 40-59: frequent wrong word choice or severe repetition
- 0-39: lexical limitations prevent the intended meaning

### Coherence

- 90-100: clear organization, reference, linking, and information flow
- 75-89: easy to follow with minor jumps
- 60-74: basic sequence but weak linking or unclear references
- 40-59: fragmented organization
- 0-39: no coherent account

### Mechanics

- 90-100: spelling and punctuation are consistently controlled
- 75-89: a few minor issues
- 60-74: recurring issues but readable
- 40-59: frequent issues interfere with reading
- 0-39: mechanics severely obstruct comprehension

### Overall

Use the weighted score:

- comprehension: 30 percent
- grammar: 20 percent
- vocabulary: 25 percent
- coherence: 15 percent
- mechanics: 10 percent

Round to the nearest integer. Do not inflate the overall score to be encouraging.

## CEFR-oriented progress model

Track dimensions separately:

1. reading comprehension and inference;
2. receptive vocabulary;
3. productive vocabulary;
4. collocation, phraseology, and register;
5. grammatical range and control;
6. coherence and discourse organization;
7. lexical and grammatical error rate;
8. ability to paraphrase and express nuance.

### Evidence thresholds

Do not create a new proficiency snapshot from a single short interaction. Use at least one of:

- 20 scored review or assessment items spanning at least 3 modes and 7 days;
- 3 completed reading check-ins with at least 150 total English words;
- one formal assessment of at least 10 items plus a completed reading check-in.

Use confidence below 0.70 when evidence is minimal or narrow. A C1 claim requires sustained evidence of productive precision, collocation/register control, complex grammar, and coherent extended production, not just vocabulary recognition.

## Mastery interpretation

A database status of `mastered` means the current scheduling policy has strong evidence for retention. It does not mean complete command in every sense or register. For a human-facing claim of robust mastery, look for:

- stage at or above the policy threshold;
- at least three consecutive scores of 3 or 4;
- productive mastery at or above 0.85;
- contextual mastery at or above 0.80;
- no recent high-confidence lapse;
- at least one productive or contextual success.