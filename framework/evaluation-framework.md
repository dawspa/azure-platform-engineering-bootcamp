# Evaluation Framework

Version: 1.0

---

# Purpose

This document defines how every AI assistant evaluates the learner throughout the bootcamp.

The evaluation process is deterministic.

It must not depend on the personality, optimism, or internal reasoning of a specific AI model.

All tutors, interviewers, reviewers, and gap analyzers MUST follow this framework.

---

# Evaluation Philosophy

The goal is NOT to measure Azure knowledge.

The goal is to measure interview readiness.

Interview readiness is determined by the learner's ability to:

- understand concepts
- reason about architecture
- make engineering decisions
- troubleshoot problems
- communicate clearly
- defend technical choices

---

# Evaluation Dimensions

Every interview answer should be evaluated across the following dimensions.

---

## 1. Conceptual Understanding

Question

Does the learner understand what this technology is and why it exists?

Score

0 — No understanding

1 — Recognizes terminology only

2 — Understands basic concepts

3 — Understands relationships

4 — Can explain concepts clearly

5 — Can teach and challenge assumptions

---

## 2. Architectural Reasoning

Question

Can the learner explain where this technology belongs within a larger architecture?

Score

0 — No architectural understanding

1 — Describes components only

2 — Understands basic architecture

3 — Explains interactions

4 — Explains trade-offs

5 — Thinks like a Platform Architect

---

## 3. Decision Making

Question

Can the learner justify engineering decisions?

Score

0 — Cannot justify decisions

1 — Makes random choices

2 — Chooses based on examples

3 — Chooses based on requirements

4 — Explains trade-offs

5 — Considers business, security and operations

---

## 4. Troubleshooting

Question

Can the learner investigate and solve realistic production problems?

Score

0 — No troubleshooting approach

1 — Random guessing

2 — Basic investigation

3 — Logical troubleshooting

4 — Structured investigation

5 — Expert diagnostic reasoning

---

## 5. Communication

Question

Can the learner explain technical concepts clearly?

Score

0 — Cannot explain

1 — Confusing explanations

2 — Understandable with assistance

3 — Clear explanations

4 — Structured explanations

5 — Executive and engineering level communication

---

## 6. Confidence

Question

Does the learner communicate with confidence while remaining intellectually honest?

Score

0 — Unable to answer

1 — Constant uncertainty

2 — Hesitant

3 — Reasonably confident

4 — Confident

5 — Confident while openly acknowledging uncertainty when appropriate

---

# Gap Severity

The reviewer must classify every identified weakness.

---

## Critical

Definition

Fundamental misunderstanding.

Examples

- Confuses services.
- Cannot explain the basic purpose.
- Uses incorrect terminology.
- Architecture is fundamentally wrong.

Required Action

Restart the module from Learn.

---

## Major

Definition

The learner understands the topic but cannot apply it.

Examples

- Poor architectural reasoning.
- Cannot troubleshoot.
- Cannot explain design choices.
- Cannot compare alternatives.

Required Action

Gap Closure.

Followed by Validation Interview.

---

## Minor

Definition

Small knowledge gaps that do not significantly affect interview performance.

Examples

- Forgot a limitation.
- Forgot a feature name.
- Minor terminology issue.
- Small implementation detail.

Required Action

Document the gap.

Continue.

---

## None

The topic is interview-ready.

---

# Quality Gates

A module CANNOT be completed if any of the following conditions are true.

---

Rule 1

Conceptual Understanding < 3

Result

FAIL

---

Rule 2

Architectural Reasoning < 3

Result

FAIL

---

Rule 3

Decision Making < 3

Result

Gap Closure required.

---

Rule 4

Troubleshooting < 2

Result

Gap Closure required.

---

Rule 5

Communication < 2

Result

Practice explanation.

Knowledge review is NOT required.

---

# Interview Readiness

The interviewer should classify the learner into one of the following categories.

---

## Level 1

Not Interview Ready

Multiple Critical gaps.

---

## Level 2

Needs Additional Preparation

Major gaps remain.

---

## Level 3

Interview Ready

No Critical gaps.

No unresolved Major gaps.

Minor gaps acceptable.

---

## Level 4

Strong Senior Candidate

Can discuss architecture comfortably.

Can defend decisions.

Can troubleshoot.

---

## Level 5

Principal-Level Discussion

Can challenge assumptions.

Explains trade-offs naturally.

Drives architectural conversations.

---

# Human Override

The learner may challenge any evaluation.

If challenged, the AI assistant MUST explain:

- why the score was assigned
- which answer influenced the score
- what would improve the score

The assistant must never answer:

"I just think so."

Every evaluation must be justified.

---

# Validation Interview

Gap Closure is never considered complete without validation.

The Validation Interview should:

- cover only identified gaps
- ask new questions
- avoid repeating previous wording
- confirm understanding

If unresolved Major or Critical gaps remain, Gap Closure continues.

---

# Module Completion

A module is complete only if:

- all Critical gaps are resolved
- all Major gaps are resolved
- Quality Gates pass
- Exit Criteria are satisfied
- Validation Interview succeeds

Minor gaps may remain.

Perfect knowledge is not required.

Interview readiness is.

---

# AI Responsibilities

Every AI role has different responsibilities.

Tutor

Teach.

Never evaluate.

---

Interviewer

Challenge.

Never teach.

---

Reviewer

Evaluate.

Never teach.

---

Gap Analyzer

Prioritize weaknesses.

Never introduce new topics.

---

# Design Principles

The framework values:

understanding over memorization

architecture over configuration

reasoning over recall

communication over perfection

consistency over intensity

interview readiness over certification

---

# Definition of Done

A learner successfully completes a module when they demonstrate consistent interview-level competence according to this framework, regardless of the AI model used for evaluation.