# Module Template Specification

Version: 1.0

---

# Purpose

This document defines the mandatory structure for every learning module generated within the Azure Platform Engineering Bootcamp.

Every module MUST follow this specification.

The AI model used to generate the content is irrelevant.

The output quality depends on compliance with this contract.

---

# Module Metadata

Every module starts with metadata.

```yaml
Title:
Domain:
Difficulty:
Estimated Duration:
Prerequisites:
Learning Objectives:
Interview Focus:
```

Example

```yaml
Title: Private Endpoints
Domain: Azure Networking
Difficulty: Intermediate
Estimated Duration: 90 minutes
Prerequisites:
  - VNets
  - Subnets
  - Azure Storage

Learning Objectives:
  - Understand Private Endpoints
  - Design secure connectivity
  - Troubleshoot DNS issues

Interview Focus:
  - Senior Platform Engineer
```

---

# Mandatory Sections

Every generated module MUST contain the following sections in the exact order.

---

## 1. Learning Goal

Explain what the learner should be capable of after completing the module.

Avoid generic statements.

Goals must be measurable.

---

## 2. Why This Exists

Explain:

- Why Microsoft introduced this service
- Which problem it solves
- What existed before
- Why previous approaches were insufficient

This section develops architectural reasoning.

---

## 3. Core Concepts

Introduce the minimal theory required.

Avoid implementation details.

Focus on understanding.

---

## 4. Architecture

Explain how the service fits into Azure architecture.

Whenever possible include ASCII diagrams.

Example:

```text
Application

↓

Private Endpoint

↓

Private DNS

↓

Azure Storage
```

---

## 5. Design Decisions

Discuss:

- When to use it
- When NOT to use it
- Alternatives
- Trade-offs

This is one of the most important sections.

---

## 6. Real World Scenario

Present a realistic enterprise scenario.

Example:

"A company is migrating 400 applications from on-premises..."

The learner should understand how the technology is applied.

---

## 7. Common Mistakes

List mistakes made by engineers.

Explain WHY they are mistakes.

---

## 8. Troubleshooting

Present several failure scenarios.

Explain:

Symptoms

Possible causes

Investigation

Resolution

---

## 9. Interview Questions

Provide questions grouped by difficulty.

### Junior

Basic understanding.

### Mid

Configuration and reasoning.

### Senior

Architecture.

Trade-offs.

Troubleshooting.

### Principal

Deep architectural discussion.

No memorization.

---

## 10. Interactive Exercise

Present one architectural challenge.

Do NOT provide the solution immediately.

Allow the learner to think first.

---

## 11. AI Tutor Prompt

Provide a prompt that turns any AI assistant into a tutor for this module.

---

## 12. AI Interview Prompt

Provide a prompt that turns any AI assistant into a technical interviewer.

The interviewer should:

- ask one question at a time
- challenge answers
- continue until the learner reaches the limit of their knowledge

---

## 13. AI Reviewer Prompt

Generate a prompt that reviews interview performance.

The reviewer should:

- identify knowledge gaps
- avoid unnecessary praise
- explain what a Principal Engineer would expect

---

## 14. Microsoft Learn References

Recommend only the official Microsoft Learn content relevant to this module.

Do not recommend unrelated documentation.

---

## 15. Exit Criteria

Provide measurable completion criteria.

Example:

✓ I can explain the difference between Private Endpoint and Service Endpoint.

✓ I can troubleshoot DNS resolution problems.

✓ I can defend architectural decisions.

---

## 16. Self Assessment

Provide a checklist.

Example

```
□ I understand the architecture.

□ I understand the trade-offs.

□ I can troubleshoot failures.

□ I can explain this topic without documentation.

□ I would feel comfortable discussing this during a technical interview.
```

---

# Writing Style

Every module MUST follow these rules.

- Explain concepts before implementation.
- Prefer reasoning over memorization.
- Focus on architecture rather than configuration.
- Avoid certification-style content.
- Avoid unnecessary history.
- Use enterprise examples.
- Prefer diagrams over long paragraphs.
- Keep the learner engaged through questions.
- Challenge assumptions.
- Teach the "why", not only the "how".

---

# Definition of Done

A module is considered complete only if:

- every mandatory section exists;
- exit criteria are measurable;
- interview questions become progressively harder;
- AI prompts are included;
- architectural reasoning is emphasized over implementation.

Any generated module that does not satisfy this specification should be considered incomplete.