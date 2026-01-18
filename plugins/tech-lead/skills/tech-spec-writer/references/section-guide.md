# Section Writing Guide

Guidelines for writing each section of the Tech Spec.

## Problem Statement

**Purpose**: Clearly articulate WHY this work is needed.

**Good Example:**
> Users currently cannot track their portfolio performance over time. They must manually calculate gains/losses by comparing current prices with their purchase history, which is error-prone and time-consuming. This leads to poor investment decisions and user frustration.

**Bad Example:**
> We need a portfolio tracking feature.

**Checklist:**
- [ ] Specific pain point identified
- [ ] User/business impact stated
- [ ] Quantifiable if possible (e.g., "affects 30% of users")

---

## Goal

**Purpose**: Define measurable success criteria.

**Good Example:**
> - Enable users to view real-time portfolio P&L with <500ms latency
> - Reduce support tickets about portfolio calculations by 50%
> - Achieve 80% user adoption within 30 days of launch

**Bad Example:**
> Make portfolio tracking better.

**Checklist:**
- [ ] Specific and measurable
- [ ] Time-bound if applicable
- [ ] Aligned with business objectives

---

## Proposed Solution

**Purpose**: High-level "what" and "why" of your approach.

**Structure:**
1. One-sentence summary
2. Key components/changes
3. Why this approach over alternatives

**Length**: 2-4 paragraphs. Save details for specific sections.

---

## Changes Summary

**Purpose**: Quick overview of all affected areas.

**Format**: Bullet list grouped by category:
- **Backend**: [changes]
- **Frontend**: [changes]
- **Database**: [changes]
- **Infrastructure**: [changes]
- **External Services**: [changes]

---

## Architectural Diagrams

**Purpose**: Visual representation of system changes.

**Include when:**
- New services are added
- Service communication patterns change
- Data flow is modified
- Third-party integrations are added

**Diagram types:**
- **C4 Context**: System boundaries and external actors
- **Component**: Internal service structure
- **Sequence**: Request/response flows
- **Data Flow**: How data moves through the system

**Use Mermaid syntax** for inline diagrams when possible.

---

## Schema Specification

**Purpose**: Document all database changes.

**Include:**
- New tables with full DDL
- Column additions/modifications
- Index changes
- Migration strategy for existing data

**Format**: Use SQL code blocks for clarity.

---

## API Specification

**Purpose**: Document all API changes.

**For each endpoint include:**
- HTTP method and path
- Description
- Authentication requirements
- Request/response schemas (JSON)
- Error responses
- Breaking change indicator

**Consider**: OpenAPI spec for complex APIs.

---

## UI Flow Diagrams

**Purpose**: Document user-facing changes.

**Include:**
- User flow (numbered steps)
- Affected screens/components
- Wireframes or mockups (links)
- Edge cases and error states

---

## Risk

**Purpose**: Identify what could go wrong and how to prevent it.

**Risk categories:**
- **Technical**: Performance, scalability, complexity
- **Dependency**: External services, team dependencies
- **Timeline**: Scope creep, resource availability
- **Security**: Data exposure, authentication

**For each risk**: Likelihood + Impact + Mitigation

---

## Alternative Solutions

**Purpose**: Show you considered other options.

**Include:**
- 2-3 alternatives considered
- Pros and cons of each
- Clear reasoning for rejection

This builds confidence in the chosen approach.

---

## Implementation Plan

**Purpose**: Break work into deliverable phases.

**Each phase should have:**
- Clear deliverable
- Task breakdown
- Dependencies
- Definition of done

**Include rollout strategy:**
- Feature flags
- Staged rollout %
- Monitoring plan
- Rollback procedure

---

## Metrics

**Purpose**: Define how success is measured.

**Types:**
- **Business metrics**: Revenue, conversion, adoption
- **Technical metrics**: Latency, error rate, throughput
- **User metrics**: NPS, task completion rate

**Each metric needs:**
- Current baseline (if exists)
- Target value
- Measurement method

---

## Software Quality Attributes

**Purpose**: Define non-functional requirements.

**Fill only relevant attributes:**
- Skip attributes that don't apply
- Be specific about requirements
- Include measurable thresholds

---

## Follow Up

**Purpose**: Track action items after spec approval.

**Include:**
- Jira ticket links when created
- Clear ownership
- Dependencies between tasks
