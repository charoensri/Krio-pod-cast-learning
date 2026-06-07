# Implementation Plan: AI Evolution Podcast Script Generator

## Overview

Implement the two-minute podcast script generator as a TypeScript pipeline. Each stage is a
discrete, testable unit that transforms the central `PodcastScript` artifact. Property-based
tests (fast-check) validate all 11 correctness properties; unit and integration tests cover
example-based scenarios. Tasks build incrementally so every step leaves the system in a
runnable state.

---

## Tasks

- [ ] 1. Project setup and core type definitions
  - Initialise a TypeScript project (`tsconfig.json`, `package.json`) with dependencies:
    `fast-check` for PBT, `jest` + `ts-jest` for test execution, and `jest-circus` as the
    test runner
  - Create the top-level directory layout:
    `src/models/`, `src/pipeline/`, `src/data/`, `src/utils/`, `tests/unit/`,
    `tests/property/`, `tests/integration/`
  - Define all shared TypeScript interfaces and enums in `src/models/`:
    `ScriptConfig`, `PodcastScript`, `Section`, `SectionHeading`, `WordCountSummary`,
    `AIMilestone`, `AITrend`, `ValidationResult`, `ValidationError`, `Tone` enum
  - Define the `ErrorCode` enum with all codes from the design:
    `WORD_COUNT_OUT_OF_RANGE`, `MISSING_SECTION`, `WRONG_SECTION_ORDER`,
    `INVALID_HEADING`, `LIST_IN_SPOKEN_SECTION`, `INVALID_STAGE_DIRECTION`,
    `READABILITY_EXCEEDED`, `SENTENCE_TOO_LONG`, `YEAR_IN_INTRO`,
    `MISSING_TEMPORAL_CONTEXT`, `MILESTONES_NOT_CHRONOLOGICAL`,
    `MISSING_OPENING_LINE`, `MISSING_TOPIC_KEYWORD`
  - Define the `MANDATORY_OPENING_LINE` constant:
    `"This is a pod cast on AI Evalution brought to you by Seri Charoensri on 7/06/2026, have fun!"`
  - _Requirements: 1.1, 1.2, 1.3, 2.1, 2.2, 9.1_

- [ ] 2. Content library (static seed data)
  - [ ] 2.1 Create `src/data/contentLibrary.ts` with the five seed milestones and four seed
    trends defined in the design
    - Export typed arrays: `AI_MILESTONES: AIMilestone[]` and `AI_TRENDS: AITrend[]`
    - Ensure milestone objects include `year`, `decade`, `label`, and `description`
    - Ensure trend objects include `label`, `plainExpl`, and optional `examples`
    - Seed milestones: Dartmouth Conference (1956), Deep Blue defeats Kasparov (1997),
      AlexNet deep learning breakthrough (2012), Transformer architecture introduced (2017),
      ChatGPT public release (2022)
    - Seed trends: Large Language Models, Generative AI, AI in Healthcare, AI in Creative Fields
    - _Requirements: 3.1, 3.2, 4.1, 4.2_

  - [ ]* 2.2 Write unit tests for content library
    - Assert that `AI_MILESTONES` contains ≥ 5 entries and each entry has all required fields
    - Assert that `AI_TRENDS` contains ≥ 4 entries and each entry has all required fields
    - Assert that milestone years are already sorted in ascending order in the seed data
    - _Requirements: 3.1, 4.1_

- [ ] 3. Section Template Engine
  - [ ] 3.1 Implement `src/pipeline/sectionTemplateEngine.ts`
    - Export `buildTemplates(config: ScriptConfig): Section[]` that returns three blank
      `Section` scaffolds (Intro, Main Content, Outro) with placeholder tokens in `body`
    - Intro template: contains `{{HOOK}}`, `{{TOPIC_STATEMENT}}`, `{{TRANSITION}}`
    - Main Content template: contains `{{MILESTONES}}`, `{{TRENDS}}`
    - Outro template: contains `{{THEME_SUMMARY}}`, `{{CALL_TO_ACTION}}`
    - _Requirements: 1.1, 1.2, 1.3_

  - [ ]* 3.2 Write unit tests for SectionTemplateEngine
    - Assert returned array has exactly three sections with headings
      `INTRO`, `MAIN_CONTENT`, `OUTRO` in that order
    - Assert section bodies contain the expected placeholder tokens
    - _Requirements: 1.1, 1.2, 1.3_

- [ ] 4. Content Assembler
  - [ ] 4.1 Implement `src/pipeline/contentAssembler.ts`
    - Export `assembleContent(sections: Section[], config: ScriptConfig): Section[]`
    - Prepend the exact `MANDATORY_OPENING_LINE` constant as the very first spoken text
      of the Intro body, before the hook and all other content (Requirements 9.1, 9.2, 9.3)
    - Select ≥ 3 milestones from the library in chronological order; include year or decade
      in each milestone sentence
    - Select ≥ 2 trends with plain-language explanations; gloss technical terms on first use
    - Replace placeholder tokens with the assembled spoken text
    - Use a happy and upbeat tone throughout; use contractions and conversational phrasing;
      keep every sentence ≤ 25 words
    - Return `MISSING_TEMPORAL_CONTEXT` or `MILESTONES_NOT_CHRONOLOGICAL` error if
      constraints are violated before returning sections
    - _Requirements: 3.1, 3.2, 3.3, 4.1, 4.2, 5.1, 5.2, 8.3, 9.1, 9.2, 9.3, 9.4_

  - [ ]* 4.2 Write unit tests for ContentAssembler
    - Assert at least 3 distinct milestone labels appear in Main Content body
    - Assert at least 2 trend labels appear in Main Content body
    - Assert each trend is followed by a plain-language explanation within the same paragraph
    - Assert the Intro hook (first non-opening-line sentence) is a question or ends with `!` or `?`
    - Assert Intro body contains a keyword: "AI", "artificial intelligence", or "intelligence"
    - Assert the Intro body begins with the exact `MANDATORY_OPENING_LINE` constant
    - _Requirements: 3.1, 4.1, 4.2, 5.1, 6.1, 6.2, 9.1_

- [ ] 5. Utility: word count and readability helpers
  - [ ] 5.1 Implement `src/utils/wordCount.ts`
    - Export `countWords(text: string): number` — splits on whitespace, ignores bracketed
      tokens when counting spoken words
    - Export `countSentences(text: string): number`
    - Export `countSyllables(text: string): number` (heuristic: count vowel groups)
    - _Requirements: 2.1, 2.3, 2.4, 5.3_

  - [ ] 5.2 Implement `src/utils/readability.ts`
    - Export `fleschKincaidGradeLevel(text: string): number` using the formula from the design
    - Export `findLongSentences(text: string, maxWords: number): string[]` — returns sentences
      exceeding `maxWords`
    - _Requirements: 5.3, 8.3_

  - [ ]* 5.3 Write unit tests for utility functions
    - Test `countWords` on known strings including strings with `[PAUSE]` tokens
    - Test `fleschKincaidGradeLevel` against at least two reference strings with known scores
    - Test `findLongSentences` detects sentences over 25 words
    - _Requirements: 2.1, 5.3, 8.3_

- [ ] 6. Checkpoint — verify pipeline scaffolding
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Word Count Validator
  - [ ] 7.1 Implement `src/pipeline/wordCountValidator.ts`
    - Export `validateWordCounts(script: PodcastScript): ValidationResult`
    - Check Intro: 30–50 words; Outro: 30–50 words; total: 250–320 words
    - Emit `WORD_COUNT_OUT_OF_RANGE` errors with section name and actual count on failure
    - _Requirements: 2.1, 2.3, 2.4, 2.5_

  - [ ]* 7.2 Write property test for word count bounds (Property 2)
    - **Property 2: Per-section and total word count bounds**
    - Use fast-check to generate random `PodcastScript` objects with varying section word
      counts; assert the validator correctly classifies in-range vs out-of-range scripts
    - **Validates: Requirements 2.1, 2.3, 2.4, 2.5**
    - Tag: `Feature: ai-evolution-podcast, Property 2: Per-section and total word count bounds`

- [ ] 8. Word Count Summary Writer
  - [ ] 8.1 Implement `src/pipeline/wordCountSummaryWriter.ts`
    - Export `appendSummary(script: PodcastScript): PodcastScript`
    - Populate `script.summaryBlock` with per-section and total word counts derived from
      `section.wordCount` fields
    - The summary block must always be present in the returned script
    - _Requirements: 2.2_

  - [ ]* 8.2 Write property test for word count summary (Property 3)
    - **Property 3: Word count summary is always present**
    - Use fast-check to generate arbitrary assembled scripts; after calling `appendSummary`,
      assert `summaryBlock` is non-null and all four fields are non-negative integers that
      sum correctly
    - **Validates: Requirements 2.2**
    - Tag: `Feature: ai-evolution-podcast, Property 3: Word count summary is always present`

- [ ] 9. Readability Checker
  - [ ] 9.1 Implement `src/pipeline/readabilityChecker.ts`
    - Export `checkReadability(script: PodcastScript): ValidationResult`
    - Compute FKGL for each section; emit `READABILITY_EXCEEDED` if FKGL > 10
    - Find all sentences > 25 words; emit `SENTENCE_TOO_LONG` for each
    - _Requirements: 5.3, 8.3_

  - [ ]* 9.2 Write property test for readability bounds (Property 8)
    - **Property 8: Readability bound per section**
    - Use fast-check to generate random section bodies at various reading levels; assert
      that scripts whose every section has FKGL ≤ 10 and no sentence > 25 words pass, and
      that at least one violation is flagged otherwise
    - **Validates: Requirements 5.3, 8.3**
    - Tag: `Feature: ai-evolution-podcast, Property 8: Readability bound per section`

- [ ] 10. Structure Validator
  - [ ] 10.1 Implement `src/pipeline/structureValidator.ts`
    - Export `validateStructure(script: PodcastScript): ValidationResult`
    - Assert exactly three sections present; emit `MISSING_SECTION` otherwise
    - Assert headings are the canonical labels; emit `INVALID_HEADING` for mismatches
    - Assert order is Intro → Main Content → Outro by array index; emit `WRONG_SECTION_ORDER`
    - Scan body text for list markers (`-`, `*`, `•`, `1.` … pattern); emit
      `LIST_IN_SPOKEN_SECTION` on detection
    - Scan Intro body for four-digit year patterns; emit `YEAR_IN_INTRO` on detection
    - Assert the Intro body starts with the exact `MANDATORY_OPENING_LINE` constant as its
      first spoken text; emit `MISSING_OPENING_LINE` if absent, altered, or preceded by
      any other spoken text
    - _Requirements: 1.1, 1.2, 1.3, 6.3, 8.1, 9.1, 9.2, 9.3_

  - [ ]* 10.2 Write property test for three-section structure invariant (Property 1)
    - **Property 1: Three-section structure invariant**
    - Use fast-check to generate random `Section[]` arrays with arbitrary lengths and
      heading permutations; assert the validator correctly accepts only arrays of exactly
      three sections in canonical order with canonical headings
    - **Validates: Requirements 1.1, 1.2, 1.3**
    - Tag: `Feature: ai-evolution-podcast, Property 1: Three-section structure invariant`

  - [ ]* 10.3 Write property test for no list markers in spoken sections (Property 9)
    - **Property 9: No list markers in spoken sections**
    - Use fast-check to generate section bodies with and without list-marker patterns;
      assert `validateStructure` emits `LIST_IN_SPOKEN_SECTION` for any body containing
      `-`, `*`, `•`, or `1.`-style list starters
    - **Validates: Requirements 8.1**
    - Tag: `Feature: ai-evolution-podcast, Property 9: No list markers in spoken sections`

  - [ ]* 10.4 Write property test for intro contains no year references (Property 6)
    - **Property 6: Intro contains no year references**
    - Use fast-check to generate Intro bodies with and without four-digit year patterns;
      assert `validateStructure` emits `YEAR_IN_INTRO` iff a year pattern is present
    - **Validates: Requirements 6.3**
    - Tag: `Feature: ai-evolution-podcast, Property 6: Intro contains no year references`

  - [ ]* 10.5 Write property test for mandatory opening line (Property 11)
    - **Property 11: Mandatory opening line is always the first spoken text**
    - Use fast-check to generate `PodcastScript` objects where the Intro body either starts
      with the exact `MANDATORY_OPENING_LINE` constant or has it missing, paraphrased, or
      preceded by other text; assert `validateStructure` emits `MISSING_OPENING_LINE` in
      all non-exact-match cases and passes only when the line is present verbatim at the start
    - **Validates: Requirements 9.1, 9.2, 9.3**
    - Tag: `Feature: ai-evolution-podcast, Property 11: Mandatory opening line is always the first spoken text`

- [ ] 11. Stage Direction Formatter
  - [ ] 11.1 Implement `src/pipeline/stageDirectionFormatter.ts`
    - Export `formatStageDirections(script: PodcastScript): ValidationResult`
    - Parse all bracketed tokens (`\[.*?\]`) in every section body
    - Allow only: `[INTRO]`, `[MAIN CONTENT]`, `[OUTRO]`, `[PAUSE]`, `[EMPHASIS]`
    - Emit `INVALID_STAGE_DIRECTION` with the offending token for any unknown token
    - _Requirements: 8.2_

  - [ ]* 11.2 Write property test for stage direction vocabulary (Property 10)
    - **Property 10: Stage directions use only the allowed vocabulary**
    - Use fast-check to generate arbitrary strings containing random bracketed tokens;
      assert `formatStageDirections` emits `INVALID_STAGE_DIRECTION` for tokens outside
      the five allowed values and passes for documents containing only allowed tokens
    - **Validates: Requirements 8.2**
    - Tag: `Feature: ai-evolution-podcast, Property 10: Stage directions use only the allowed vocabulary`

- [ ] 12. Milestone temporal context and ordering validators
  - [ ] 12.1 Implement temporal context checks in `src/pipeline/contentAssembler.ts`
    (extend existing module)
    - Export `validateMilestoneContext(mainContentBody: string, milestones: AIMilestone[]): ValidationResult`
    - For each milestone label present in the body, assert a year (`\d{4}`) or decade
      (`\d{4}s`) appears in the same or adjacent sentence; emit `MISSING_TEMPORAL_CONTEXT`
      otherwise
    - Extract all year/decade references in order; assert strictly ascending; emit
      `MILESTONES_NOT_CHRONOLOGICAL` on failure
    - _Requirements: 3.2, 3.3_

  - [ ]* 12.2 Write property test for milestone temporal context (Property 4)
    - **Property 4: Milestone temporal context**
    - Use fast-check to generate Main Content bodies with milestone labels present with and
      without adjacent year/decade tokens; assert `validateMilestoneContext` correctly
      emits `MISSING_TEMPORAL_CONTEXT` when temporal context is absent
    - **Validates: Requirements 3.2**
    - Tag: `Feature: ai-evolution-podcast, Property 4: Milestone temporal context`

  - [ ]* 12.3 Write property test for chronological milestone ordering (Property 5)
    - **Property 5: Milestones appear in chronological order**
    - Use fast-check to generate sequences of year references in random order; assert
      `validateMilestoneContext` emits `MILESTONES_NOT_CHRONOLOGICAL` iff the sequence is
      not strictly ascending
    - **Validates: Requirements 3.3**
    - Tag: `Feature: ai-evolution-podcast, Property 5: Milestones appear in chronological order`

- [ ] 13. Checkpoint — validate all pipeline stages individually
  - Ensure all tests pass for tasks 7–12, ask the user if questions arise.

- [ ] 14. Intro topic-keyword validator
  - [ ] 14.1 Add `validateIntroTopic(introBody: string): ValidationResult` to
    `src/pipeline/structureValidator.ts`
    - Check that at least one of the first three sentences (excluding the mandatory opening
      line) contains "AI", "artificial intelligence", or "intelligence" (case-insensitive)
    - Emit `MISSING_TOPIC_KEYWORD` error code if not found
    - _Requirements: 6.2_

  - [ ]* 14.2 Write property test for topic stated in Intro opening (Property 7)
    - **Property 7: Topic stated in the Intro opening**
    - Use fast-check to generate Intro bodies with and without topic keywords in the first
      three sentences (after the mandatory opening line); assert `validateIntroTopic` passes
      iff at least one keyword is present in that range
    - **Validates: Requirements 6.2**
    - Tag: `Feature: ai-evolution-podcast, Property 7: Topic stated in the Intro opening`

- [ ] 15. Pipeline orchestrator
  - [ ] 15.1 Implement `src/pipeline/generator.ts`
    - Export `generateScript(config: ScriptConfig): PodcastScript | ValidationResult`
    - Wire all pipeline stages in order:
      1. `buildTemplates`
      2. `assembleContent` (includes mandatory opening line prepend)
      3. `validateMilestoneContext` — halt on error
      4. `validateWordCounts` — halt on error
      5. `checkReadability` — halt on error
      6. `validateStructure` (includes opening line and list checks) — halt on error
      7. `validateIntroTopic` — halt on error
      8. `formatStageDirections` — halt on error
      9. `appendSummary`
    - On any stage error, return the `ValidationResult` immediately (no partial delivery)
    - On success, return the fully validated `PodcastScript`
    - _Requirements: 1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 2.4, 2.5, 3.1, 3.2, 3.3, 4.1, 4.2,
      5.1, 5.3, 6.1, 6.2, 6.3, 7.1, 7.2, 8.1, 8.2, 8.3, 9.1, 9.2, 9.3_

  - [ ]* 15.2 Write unit tests for Outro requirements
    - Assert Outro body contains at least one imperative or CTA phrase (Requirement 7.2)
    - Assert Outro body contains at least one sentence summarising the episode theme
      (Requirement 7.1)
    - _Requirements: 7.1, 7.2_

- [ ] 16. Integration tests
  - [ ] 16.1 Write end-to-end integration test with default `ScriptConfig`
    - Call `generateScript({})` and assert the result is a `PodcastScript` (not a
      `ValidationResult`)
    - Assert all three sections are present in order with canonical headings
    - Assert total word count is 250–320, Intro 30–50, Outro 30–50
    - Assert `summaryBlock` is populated and values match section word counts
    - Assert the Intro body starts with the exact `MANDATORY_OPENING_LINE` string
    - Assert at least 3 milestone references appear in Main Content
    - Assert at least 2 trend references appear in Main Content
    - _Requirements: 1.1, 1.2, 1.3, 2.1, 2.2, 2.3, 2.4, 2.5, 3.1, 4.1, 9.1, 9.2, 9.3_

  - [ ] 16.2 Write end-to-end integration test with `topicHints` override
    - Call `generateScript({ topicHints: ["autonomous vehicles", "AI safety"] })` and assert
      the result satisfies all structural constraints (same assertions as 16.1)
    - Assert the Intro body still starts with the exact `MANDATORY_OPENING_LINE` string
    - _Requirements: 1.1, 2.1, 2.3, 2.4, 2.5, 9.1, 9.2, 9.3_

- [ ] 17. Final checkpoint — all tests green
  - Ensure all tests pass, ask the user if questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP build.
- Each task references specific requirements for full traceability.
- The pipeline is fail-fast: a `ValidationResult` with errors short-circuits further stages.
- Property tests use fast-check with ≥ 100 iterations per property.
- Tag format for every PBT test: `Feature: ai-evolution-podcast, Property {N}: {property_text}`
- Correctness properties 1–11 each map to exactly one `it()`/`test()` block in `tests/property/`.
- Requirements 5.2 (conversational tone) and 7.3 (Outro introduces no new facts) are
  intentionally excluded from automated tests and require editorial review per the design.
- Requirement 9.4 (happy and upbeat tone) is also excluded from automated tests — it is a
  subjective quality enforced through the content authoring guidelines in `ContentAssembler`
  and requires editorial review.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "3.1", "5.1", "5.2"] },
    { "id": 2, "tasks": ["2.2", "3.2", "4.1", "5.3"] },
    { "id": 3, "tasks": ["4.2", "7.1", "8.1", "9.1", "10.1", "11.1", "12.1"] },
    { "id": 4, "tasks": ["7.2", "8.2", "9.2", "10.2", "10.3", "10.4", "10.5", "11.2", "12.2", "12.3", "14.1"] },
    { "id": 5, "tasks": ["14.2", "15.1"] },
    { "id": 6, "tasks": ["15.2", "16.1", "16.2"] }
  ]
}
```
