<!--
Sync Impact Report
Version change: 1.0.0 → 1.1.0
Modified principles:
- I. Domain-Centered MVC Architecture → I. Feature-Centered Architecture
- II. Protect Domain and Data Integrity → II. Consistent, Documented API Contracts
- III. Verify Behavior with Automated Tests → III. Validated Domain and API Data
- IV. Preserve Supported Database Compatibility → IV. Tested Behavior and Database Compatibility
- V. Secure and Operable by Default → V. Secure, Observable, Maintainable Systems
Added sections: None
Removed sections: None
Follow-up TODOs: Confirm and replace TODO(RATIFICATION_DATE) with the original adoption date.
-->

# Spring PetClinic Constitution

## Core Principles

### I. Feature-Centered Architecture
Organize application code by business feature and keep related web, domain, and persistence
responsibilities cohesive. Preserve the server-rendered Spring MVC and Thymeleaf interface;
do not replace it with a REST-first or single-page architecture without explicit approval.
Introduce service abstractions when they own meaningful use-case logic or coordination, not
when they only forward calls. This keeps changes understandable and avoids unnecessary
layers.

### II. Consistent, Documented API Contracts
Any HTTP API MUST follow REST conventions, use explicit API versioning, and return
standardized error responses. API contracts MUST be documented with OpenAPI 3 and kept
consistent with implementation. These requirements apply to APIs and do not require the
existing server-rendered MVC interface to become a REST API.

### III. Validated Domain and API Data
Validate untrusted input at application boundaries with Jakarta Validation and enforce
domain invariants before persisting changes. Changes to owners, pets, visits, veterinarians,
or specialties MUST preserve their relationships and existing data semantics. Use Java
records for data-transfer objects when their immutable data-carrier semantics fit the
contract; use domain entities for persistence and domain behavior.

### IV. Tested Behavior and Database Compatibility
Every behavior change MUST include or update automated tests at the narrowest effective
level. Use focused tests for domain, validation, and controller behavior; use integration
tests for persistence, API contracts, database-specific behavior, and application wiring.
The application MUST continue to support H2, MySQL, and PostgreSQL. Changes to entity
mappings, queries, or persistence behavior MUST be verified against affected database
profiles. Database-specific SQL MUST NOT remove supported database behavior without an
explicitly approved compatibility change.

### V. Secure, Observable, Maintainable Systems
Do not commit credentials or other secrets; keep environment-specific configuration
externalized. Validate untrusted input, rely on framework output escaping for rendered user
data, and avoid exposing sensitive details in errors or logs. Use Spring Boot Actuator and
Micrometer for operational health and metrics, configured to avoid exposing sensitive
information. Keep types focused, dependencies explicit, and designs maintainable; apply
SOLID principles when they improve separation of responsibilities and testability without
adding unnecessary abstraction.

## Additional Constraints

Java 21 LTS is mandatory. The application uses Spring Boot, Spring MVC, Spring Data JPA,
and Thymeleaf; its user interface remains server-rendered unless an architecture change is
explicitly approved. H2 is the default database, with MySQL and PostgreSQL supported
through Spring profiles. New dependencies and infrastructure MUST be justified by a
concrete requirement and kept to the smallest effective scope.

## Development Workflow

For non-trivial feature work, define behavior and acceptance criteria in the Spec Kit
artifacts before implementation. Keep implementation, tests, API documentation, and
directly affected documentation consistent. Before merge, run relevant Maven tests and
project validation checks with Java 21. Changes affecting API contracts MUST update and
validate their OpenAPI 3 documentation and API tests; changes affecting supported database
behavior MUST include suitable integration coverage. Reviewers MUST check that the change
follows this constitution, and any intentional exception MUST be documented with its
rationale and approval.

## Governance

This constitution governs project development and takes precedence over conflicting local
conventions. Amendments MUST be proposed with their rationale, reviewed and approved through
the project's normal change-review process, and recorded in this file. An amendment MUST
update the version and last-amended date.

Use semantic versioning for this constitution: MAJOR for backward-incompatible changes to
principles or governance, MINOR for new principles or materially expanded requirements, and
PATCH for clarifications and non-semantic wording changes. During implementation and review,
authors and reviewers MUST check applicable principles, tests, compatibility requirements,
API documentation, and documented exceptions. The original ratification date is not
available in the repository and requires confirmation.

**Version**: 1.1.0 | **Ratified**: TODO(RATIFICATION_DATE): confirm original adoption date | **Last Amended**: 2026-11-20
