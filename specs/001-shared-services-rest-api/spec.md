# Feature Specification: Shared Services and REST API

**Feature Branch**: `001-shared-services-rest-api`

**Created**: 2026-11-20

**Status**: Draft

**Input**: User description: "Refactor the Spring Boot application to support both the existing MVC UI and new REST APIs. Keep all current functionality and user workflows unchanged.
Introduce a shared service layer used by both MVC and API controllers, exposing the same business capabilities through REST endpoints. This will enable a gradual migration to a React, Angular, or other JavaScript frontend.
The long-term goal is to replace the MVC UI entirely and move to an API-driven architecture without a big-bang rewrite."

## Clarifications

### Session 2026-11-20

- Q: Should the new REST API require authentication, or remain available under the application's existing access policy? → A: Keep the existing access policy; do not introduce a new authentication requirement for the REST API.
- Q: How should clients select an API version when calling the new REST endpoints? → A: Include the version in the URL.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Continue using the clinic UI (Priority: P1)

Clinic staff continue managing owners, pets, visits, and veterinarian information through
the existing browser-based interface while the new API is introduced.

**Why this priority**: Keeping current staff workflows uninterrupted is a firm constraint
and is required for a gradual, rather than all-at-once, migration.

**Independent Test**: Complete the existing owner, pet, visit, and veterinarian-list
workflows through the browser and compare their outcomes with the current behavior.

**Acceptance Scenarios**:

1. **Given** existing clinic data, **When** staff search for and view owners, **Then** the
   results, details, and navigation behave as they do today.
2. **Given** valid owner or pet information, **When** staff create or update an owner or
   add or edit a pet, **Then** the existing browser workflow succeeds and saves the same
   information as before.
3. **Given** a pet eligible for a visit, **When** staff book a visit through the browser,
   **Then** the visit is saved and displayed in the existing workflow.
4. **Given** clinic veterinarians, **When** staff open the veterinarian list, **Then** the
   existing list and pagination remain available.
5. **Given** a consumer of the existing veterinarian data resource, **When** it requests
   veterinarian information, **Then** the current resource remains compatible.

---

### User Story 2 - Use clinic capabilities from another frontend (Priority: P2)

A JavaScript frontend or another API consumer can perform the same supported clinic
operations without depending on server-rendered pages.

**Why this priority**: Exposing current business capabilities through a stable interface
allows a new frontend to be adopted incrementally.

**Independent Test**: Use an API client to search, create, retrieve, and update owners; add
and edit pets using available pet types; book a visit; and retrieve the veterinarian list.
Verify saved data and validation outcomes match the browser interface.

**Acceptance Scenarios**:

1. **Given** clinic records, **When** an API consumer searches for owners, **Then** it can
   retrieve matching owners and their details, including associated pets and visits where
   available.
2. **Given** valid owner or pet data and the available pet types, **When** an API consumer
   creates or updates an owner or adds or edits a pet, **Then** the change is saved and
   reflected in both interfaces.
3. **Given** a valid visit request for a pet, **When** an API consumer books the visit,
   **Then** it is saved and visible in the browser interface.
4. **Given** veterinarian records, **When** an API consumer requests the veterinarian
   list, **Then** it receives the available veterinarian information.
5. **Given** pet types configured for the clinic, **When** an API consumer requests
   available pet types, **Then** it can use them when adding or editing a pet.
6. **Given** an invalid request or a record that does not exist, **When** an API consumer
   makes the corresponding request, **Then** it receives a standardized error response
   that identifies the problem without exposing sensitive details.

---

### User Story 3 - Migrate frontend workflows incrementally (Priority: P3)

The product team can replace browser workflows one at a time with a JavaScript frontend,
while staff can continue using workflows that have not yet moved.

**Why this priority**: Incremental migration reduces disruption and avoids requiring a
single release to replace the entire user interface.

**Independent Test**: Complete a workflow through the new API-based frontend, then complete
another unchanged workflow through the existing browser interface; confirm both use the
same clinic records and rules.

**Acceptance Scenarios**:

1. **Given** the existing UI and the new API are both available, **When** staff perform
   related operations through either interface, **Then** both interfaces show consistent
   saved data and business outcomes.
2. **Given** only some workflows have moved to a new frontend, **When** staff use a
   workflow that has not moved, **Then** it remains available through the existing UI.
3. **Given** the team releases an API-based workflow, **When** it is used alongside the
   existing UI, **Then** the UI is not required to be removed or disabled.

### Edge Cases

- An API request references an owner or pet that does not exist, or a pet that does not
  belong to the specified owner; the request fails with a standardized error and changes
  no unrelated records.
- An API request contains missing or invalid required fields, a duplicate pet name for the
  same owner, or a visit date that violates existing rules; validation behavior matches
  the existing application.
- Owner searches return no matches, one match, or multiple matches; API results accurately
  represent each case and support the existing paginated search capability.
- Requests use an unsupported API version or malformed input; the API returns a clear,
  standardized error and does not change data.
- Multiple interfaces update related records in sequence; each subsequent read reflects
  the latest successfully saved values.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST retain the existing browser-based workflows for owner
  search, creation, update, and details; pet addition and editing; visit booking; and
  veterinarian listing. It MUST NOT remove or alter other existing application
  functionality as a side effect of adding the APIs.
- **FR-002**: The system MUST expose through REST APIs the same business capabilities
  available in those supported workflows; it MUST NOT introduce unsupported operations
  such as deleting clinic records as part of this feature.
- **FR-003**: The system MUST allow API consumers to search for and retrieve owners,
  create and update owners, retrieve available pet types, add and edit their pets, book
  pet visits, and retrieve veterinarians.
- **FR-004**: The system MUST apply the same validation rules, domain rules, and
  persistence outcomes to equivalent operations through the browser and API interfaces.
- **FR-005**: The system MUST preserve existing search, pagination, relationship, and
  validation behavior, including owner-pet relationships, duplicate pet-name handling,
  and visit-date constraints.
- **FR-006**: The browser and REST interfaces MUST use the same business capabilities so
  that a change made through one interface is visible through the other.
- **FR-007**: New REST endpoints MUST include the API version in the URL, follow REST
  conventions, and return standardized error responses for invalid requests, missing
  records, and unsupported versions. The existing veterinarian data resource remains
  subject to FR-011.
- **FR-008**: The system MUST provide OpenAPI 3 documentation for new REST endpoints and
  document the compatibility behavior of the existing veterinarian data resource. The
  documentation MUST match request, response, validation, and error behavior.
- **FR-009**: The REST APIs MUST support replacing workflows incrementally; introducing
  the APIs MUST NOT require removal or disabling of the existing browser interface.
- **FR-010**: API requests MUST follow the application's existing security and
  configuration boundaries, MUST NOT introduce a new authentication requirement, and
  MUST NOT expose sensitive details in responses.
- **FR-011**: The existing veterinarian data resource MUST remain compatible for current
  consumers while versioned REST endpoints are introduced.

### Key Entities

- **Owner**: A clinic client with personal and contact information and associated pets.
- **Pet**: An animal belonging to an owner, with a name, birth date, type, and visit
  history.
- **Visit**: A dated appointment associated with a pet and its owner.
- **Veterinarian**: A clinic professional whose information is available to staff.
- **Pet type**: A category used to describe pets.
- **API contract**: The documented, versioned requests, responses, and errors used by
  frontend applications and other API consumers.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All existing owner, pet, visit, and veterinarian-list browser workflows and
  the existing veterinarian data resource pass regression checks with no user-visible or
  consumer-visible behavior removed or changed.
- **SC-002**: API clients can complete every operation listed in FR-003, and each
  operation's saved results are visible through the browser interface.
- **SC-003**: For every supported operation available in both interfaces, equivalent valid
  inputs produce equivalent saved business data and equivalent validation outcomes.
- **SC-004**: Every published REST endpoint and its success and error responses are
  represented in the OpenAPI 3 documentation.
- **SC-005**: The existing browser interface and API remain usable together throughout
  migration; replacing one workflow does not require replacing the others.

## Assumptions

- Existing browser behavior is the source of truth for supported business operations and
  validation rules; this feature does not add delete or other currently unsupported
  operations.
- JavaScript frontend implementation is not part of this feature; the APIs must enable a
  frontend to be introduced in a later, separately planned change.
- Authentication and authorization policy remain unchanged from the application's
  existing access policy; this feature does not independently redefine access policy.
- The implementation will introduce the user-requested shared service layer so both
  browser and REST controllers use the same business capabilities.
