# Implementation Plan: Shared Services and REST API

**Branch**: `001-shared-services-rest-api` | **Date**: 2026-11-20 | **Spec**:
[spec.md](spec.md)

**Input**: Feature specification from `specs/001-shared-services-rest-api/spec.md`

## Summary

Extract owner, pet, visit, pet-type, and veterinarian business operations into
feature-centered application services used by both the existing MVC controllers and new
REST controllers. Add versioned `/api/v1` endpoints with dedicated record-based DTOs,
standardized Problem Details errors, and OpenAPI 3 documentation. Preserve all HTML routes
and workflows, as well as the existing `GET /vets` JSON wrapper, during incremental
frontend migration.

## Technical Context

**Language/Version**: Java 21 LTS. The constitution requires Java 21; both build files and
CI currently target Java 17 and must be aligned as a prerequisite.

**Primary Dependencies**: Spring Boot 4.1.0, Spring MVC, Spring Data JPA, Jakarta
Validation, Thymeleaf, and `springdoc-openapi-starter-webmvc-ui` 3.2.1 for OpenAPI 3 and
Swagger UI. Add the OpenAPI dependency consistently to Maven and Gradle.

**Storage**: Existing JPA persistence; H2 by default, with MySQL and PostgreSQL supported.
No schema change is required.

**Testing**: JUnit 5, AssertJ, Spring MVC MockMvc, Spring Boot integration tests,
Testcontainers for database-specific coverage; run Maven and Gradle builds.

**Target Platform**: Existing Spring Boot web application on Java 21.

**Project Type**: Server-rendered web application with a co-located versioned REST API.

**Performance Goals**: No new latency or throughput target is specified. Preserve the
existing application behavior and avoid unbounded result sets by retaining pagination.

**Constraints**: Keep existing UI routes, forms, redirects, validation, data, and
`GET /vets` response compatible. New endpoints use `/api/v1`, existing access policy,
Jakarta Validation, OpenAPI 3, and shared business services. Do not expose JPA entities
directly from the REST API.

**Scale/Scope**: Existing clinic records and user workflows; API reads and writes cover
owners, pets, visits, pet types, and veterinarians. Owner search remains last-name prefix
search with one-based pages and a default page size of five; cap requested page size at
100.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle or constraint | Plan alignment |
|---|---|
| I. Feature-Centered Architecture | Place application services and API adapters with their owner, pet, visit, pet-type, and vet features; keep view rendering and HTTP presentation in their respective controllers. |
| II. Consistent, Documented API Contracts | Publish new endpoints under `/api/v1`, document contracts in OpenAPI 3, and retain the legacy `/vets` resource. |
| III. Validated Domain and API Data | Validate API DTOs at request boundaries, enforce shared domain rules in services, and use records for immutable API DTOs. |
| IV. Tested Behavior and Database Compatibility | Preserve MVC regression tests; add service, API contract, and cross-interface tests; validate persistence changes against supported databases. |
| V. Secure, Observable, Maintainable Systems | Reuse current access policy, externalized configuration, safe error responses, and existing Actuator/Micrometer instrumentation. |
| Java 21 LTS | Existing Maven, Gradle, and CI configuration targets Java 17. Align both build descriptors and both CI workflows to Java 21 as a prerequisite; verify local toolchain. |

**Gate result**: PASS with the Java 21 alignment explicitly included in the feature
implementation. No database migration or security-policy change is required.

## Project Structure

### Documentation (this feature)

```text
specs/001-shared-services-rest-api/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── openapi.yaml
```

### Source Code (repository root)

```text
src/main/java/org/springframework/samples/petclinic/
├── owner/
│   ├── OwnerController.java
│   ├── PetController.java
│   ├── VisitController.java
│   ├── OwnerService.java
│   ├── PetService.java
│   ├── PetTypeService.java
│   ├── VisitService.java
│   └── api/
│       ├── OwnerApiController.java
│       ├── PetApiController.java
│       ├── PetTypeApiController.java
│       ├── VisitApiController.java
│       └── dto/
├── api/
│   ├── ApiExceptionHandler.java
│   ├── ApiProblem.java
│   └── PageResponse.java
├── vet/
│   ├── VetController.java
│   ├── VetService.java
│   └── api/
│       ├── VetApiController.java
│       └── dto/

src/test/java/org/springframework/samples/petclinic/
├── owner/                 # Existing MVC and domain regression tests
├── owner/api/             # REST endpoint and error contract tests
├── api/                   # Shared API errors and OpenAPI tests
├── service/               # Shared-service behavior tests
└── vet/                   # Legacy and versioned veterinarian contract tests
```

Build and CI configuration is shared by the same application:

```text
pom.xml
build.gradle
.github/workflows/maven-build.yml
.github/workflows/gradle-build.yml
```

**Structure Decision**: Keep the current single Spring Boot application and organize new
services and HTTP adapters by business feature. The API layer uses distinct DTOs and
controllers; it does not create a separate frontend or backend project.

## Design Decisions

- Services own shared use-case behavior, transactions, repository access, and domain
  validation that currently lives in controllers. MVC controllers retain view selection,
  model binding, redirects, and flash messages.
- Pet creation and update services enforce owner-scoped name uniqueness and translate
  duplicate-name persistence conflicts into the same business outcome for both adapters.
- REST request validation occurs at the API boundary, with business rules enforced again
  through the same services used by MVC. API DTOs are records and never serialize JPA
  relationships directly.
- New routes use `/api/v1`; the existing unversioned `GET /vets` remains an explicit
  compatibility exception and returns the existing `Vets.vetList` wrapper unchanged.
- Owner search uses a one-based `page`, default `size=5`, and `lastName` prefix query.
  API searches return an empty page for no matches; unlike the UI, they do not redirect
  or use a form error for this condition. Cap `size` at 100.
- A create-pet request requires a pet type. An update-pet request may omit the type to
  retain its existing value, matching the existing form-binding behavior.
- Invalid API pagination (`page < 1`, `size < 1`, or `size > 100`) returns a standardized
  400 response; the existing UI keeps its current out-of-range page redirect behavior.
- Use RFC 9457 Problem Details (`application/problem+json`) for API errors, adding a
  stable application `code` and a list of field-level validation errors where applicable.
  Do not include exception messages or other sensitive internals.
- Add the same Springdoc WebMVC UI dependency version to Maven and Gradle. Check in the
  reviewed OpenAPI document as the API contract and verify runtime-generated documentation
  matches the endpoints.
- Update Java configuration from 17 to 21 in both build descriptors and both CI workflows
  to satisfy the existing constitution.

## Complexity Tracking

No constitution violations or additional projects are introduced.
