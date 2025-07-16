# JMS Architectural Decision Records (ADRs)

This repository contains all Architectural Decision Records for the JMS (Jury Management System) project.

## What are ADRs?

An Architectural Decision Record (ADR) is a document that captures an important architectural decision made along with its context and consequences.

## ADR Format

We use the following format for our ADRs:

```markdown
# ADR-XXXX: Title

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-YYYY]

## Context
What is the issue that we're seeing that is motivating this decision or change?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or more difficult to do because of this change?
```

## ADR Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [ADR-0001](adr/ADR-0001-use-adr.md) | Use Architectural Decision Records | Accepted | 2024-01-15 |

## Creating a New ADR

1. Copy the template from `template/adr-template.md`
2. Name it `adr/ADR-XXXX-brief-description.md` (increment XXXX)
3. Fill in all sections
4. Create a pull request
5. Update the index in this README

## Categories

- **Architecture**: Overall system design decisions
- **Security**: Security-related architectural decisions
- **Infrastructure**: Infrastructure and deployment decisions
- **Data**: Data storage, encryption, and management decisions
- **Integration**: Third-party service integration decisions

## For LLM Engineers

When referencing ADRs in implementation:
- Always cite the ADR number in code comments
- Link to relevant ADRs in pull requests
- Update ADRs if implementation reveals new information