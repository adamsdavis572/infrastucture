# Copilot Instructions for Spec Driven Development (SDD)

These instructions guide Copilot in assisting with **Spec Driven Development (SDD)**, inspired by the Kiro framework. The goal is to generate structured requirements, design documentation, and actionable tasks, while maintaining robust traceability and source control practices.

---

## General Guidelines

* **Context Awareness**: Understand that the primary focus is on defining software behaviour through specifications before implementation. The **collective set of requirements, design, and task documents form the complete specification**.
* **Structured Output**: Prioritise clear, concise, and structured output formats.
* **UK English**: All output should be in UK English.

---

## Key Principles for Copilot

* **Structured Thinking**: Always strive for a structured approach to defining features, designs, and tasks.
* **Precision**: Be precise in language, avoiding ambiguity.
* **Completeness**: Aim to cover all aspects of a requirement, design, or task as requested, contributing to the overall specification.
* **Consistency**: Maintain consistent formatting and terminology across all generated files.

---

## Core SDD Documentation Files

These are the three primary documentation files that form the complete specification through custom chat modes:

### 1. Requirements Document (`requirements.md`)
* **Purpose**: Defines what the system must do from a user and business perspective
* **Location**: Project root or `.kiro/` directory
* **Structure**: Each requirement must have a unique `REQ-XXX` identifier
* **Content**: User stories, acceptance criteria, business rules, and functional/non-functional requirements

### 2. Design Document (`design.md`)
* **Purpose**: Defines how the system will be architected and implemented
* **Location**: Project root or `.kiro/` directory  
* **Structure**: Technical architecture, API specifications, data models, and component designs
* **Content**: System architecture, interface definitions, technology choices, and implementation patterns

### 3. Tasks Document (`tasks.md`)
* **Purpose**: Breaks down requirements and design into actionable development tasks
* **Location**: Project root or `.kiro/` directory
* **Structure**: Each task must have a unique `TASK-XXX` identifier
* **Content**: Development tasks, acceptance criteria, dependencies, and effort estimates

**Traceability**: These documents must maintain clear links between requirements (`REQ-XXX`), design decisions, and implementation tasks (`TASK-XXX`).

---

## Project Steering Files (Context)

These files provide foundational, project-specific context that guides all Copilot interactions. They should be located in the `.kiro/steering/` directory.

* [Product Vision](.kiro/steering/product.md)
* [Technical Stack](.kiro/steering/tech.md)
* [Project Structure](.kiro/steering/structure.md)

---

## Source Control and Traceability Guidance

This section guides Copilot on best practices for using Git to ensure traceability between requirements, tasks, and code.

### 1. Unique Identifiers as Traceability Keys

* **Requirement IDs**: Use `REQ-XXX` (e.g., `REQ-001`) as defined in `requirements.md`.
* **Task IDs**: Use `TASK-XXX` (e.g., `TASK-001`) as defined in `tasks.md` or from an external issue tracker (e.g., `PROJ-123` from Jira).

### 2. Commit Message Structure

When generating commit messages, always ensure they are clear, concise, and include relevant traceability IDs.

* **Format**: `[type]: [short description] ([ID-1], [ID-2], ...)`
    * `type` can be `feat` (new feature), `fix` (bug fix), `docs` (documentation), `style` (formatting), `refactor` (code refactoring), `test` (adding tests), `chore` (maintenance).
* **Guidance**:
    * **Link to Tasks**: Every code change should relate to at least one **Task ID**.
    * **Link to Requirements**: If the commit directly implements or significantly impacts a requirement, include its **Requirement ID**.
    * **Granular Commits**: Encourage small, atomic commits that address a single logical change. This makes traceability clearer.

**Copilot's Role for Commit Messages**:
* When suggesting code changes or completions, prompt the user to consider the relevant Task ID for the commit message.
* When asked to summarise changes for a commit message, generate a message following the specified structure and suggest relevant IDs based on the current context (open requirements, active tasks).

### 3. When to Commit

Encourage commits at logical checkpoints:

* When a specific part of a task is completed.
* After implementing a small, isolated feature.
* After fixing a single bug.
* Before starting a new, distinct piece of work.
* Before switching branches or contexts.
* After generating or updating documentation related to requirements or design.

**Copilot's Role for Commit Frequency**:
* If a substantial block of code or documentation has been generated or modified, suggest committing the changes.
* Prompt the user to ensure documentation (requirements, design, tasks) is committed alongside related code changes.

### 4. Pull Request Descriptions

While Copilot may not directly create pull requests, it should generate information that can populate them.

* **Guidance**: Pull request descriptions should summarise the changes and explicitly list all **Task IDs** and **Requirement IDs** addressed by the PR.

**Copilot's Role for Pull Requests**:
* When asked to summarise work done for a feature or task, provide a summary that can serve as a pull request description, including linked IDs.

