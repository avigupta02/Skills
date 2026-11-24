---
name: BDDFeature
description: Generate high-quality, framework-independent Gherkin BDD feature files from user stories, acceptance criteria, business requirements, and functional specifications.
---

# BDDfeature

## Purpose

Generate clear, structured, maintainable, and executable BDD feature files using Gherkin syntax.

This skill helps BDD Generation Agents translate business requirements into testable scenarios that describe expected system behavior.

## Inputs

The skill accepts one or more of the following:

- Jira user story.
- User story description.
- Acceptance criteria.
- Business requirements.
- Functional specifications.
- Existing application behavior.
- API or UI functional requirements.

## Instructions

When generating a feature file, follow these rules.

### 1. Understand the Requirement

- Read the complete user story and acceptance criteria.
- Identify the actor, business objective, preconditions, actions, and expected outcomes.
- Identify positive, negative, boundary, and validation scenarios.
- Identify dependencies and business rules.
- Do not invent requirements or expected behavior.
- If critical information is missing, explicitly identify assumptions or request clarification.

### 2. Feature File Structure

Use the standard Gherkin structure:

- Feature
- Background (only when common setup is required)
- Scenario
- Scenario Outline (when the same behavior needs multiple data combinations)
- Examples (for Scenario Outlines)

### 3. Scenario Generation Rules

Generate scenarios covering applicable cases:

- Happy path / successful execution.
- Negative scenarios.
- Input validation.
- Boundary conditions.
- Business rule validation.
- Authorization and access control.
- Error handling.
- Empty or missing data.
- Duplicate submissions or operations.
- Integration failures, where applicable.

Avoid unnecessary or duplicate scenarios.

### 4. Gherkin Syntax Rules

Every scenario must follow:

- Given: Initial context or preconditions.
- When: User action or system event.
- Then: Expected observable outcome.
- And / But: Additional conditions or outcomes.

Use the following guidelines:

- Write scenarios in simple, business-readable language.
- Use declarative steps rather than implementation details.
- Avoid UI selectors, CSS identifiers, database queries, and programming code.
- Avoid combining multiple unrelated actions in one step.
- Make expected outcomes explicit and testable.
- Use consistent terminology throughout the feature file.
- Use Scenario Outline and Examples for data-driven scenarios where appropriate.

### 5. Naming Conventions

- Feature names must describe business capabilities.
- Scenario names must describe the behavior being validated.
- Use meaningful and unique scenario names.
- Prefer business terminology over technical terminology.

### 6. Output Requirements

Generate a valid `.feature` file.

Use this template:

```gherkin
@feature-tag
Feature: [Business capability]

  Background:
    Given [common precondition]

  @positive
  Scenario: [Successful business scenario]
    Given [initial context]
    When [user performs an action]
    Then [expected outcome]

  @negative
  Scenario: [Failure or validation scenario]
    Given [initial context]
    When [user performs an invalid action]
    Then [expected error or behavior]
```

The Background section is optional and should only be included when shared setup genuinely exists.

Tags are optional. Use meaningful tags when they improve test organization.

### 7. Quality Validation

Before producing the final feature file, verify:

- All acceptance criteria are covered by one or more scenarios.
- Scenarios are independent wherever possible.
- Each scenario has a clear expected outcome.
- Given, When, and Then are used correctly.
- No duplicate scenarios exist.
- Gherkin syntax is valid.
- No unsupported business assumptions have been introduced.
- Scenario names are meaningful.
- Data-driven scenarios use appropriate Examples tables.

### 8. Output Format

Return:

1. Feature file name.
2. Complete Gherkin feature file content.
3. Acceptance criteria coverage summary.
4. Any assumptions or missing requirements.

The primary deliverable must be the `.feature` file.

Do not generate step definitions, automation code, test runners, or implementation-specific code unless explicitly requested.

## Important Constraints

- Remain technology and framework independent.
- Do not assume Selenium, Playwright, SpecFlow, Cucumber, or any particular testing framework.
- Do not modify the original business requirements.
- Never fabricate acceptance criteria.
- Prefer fewer meaningful scenarios over excessive redundant scenarios.
- Generate feature files that can be maintained by both business analysts and developers.
