# Tasks: Shared Services and REST API

**Input**: Design documents from `specs/001-shared-services-rest-api/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, and
`contracts/openapi.yaml`

**Tests**: Automated tests are included because the specification requires regression
coverage and measurable validation of equivalent MVC and API behavior.

**Organization**: Tasks are grouped by user story so each increment can be implemented
and verified in priority order.

## Phase 1: Setup

**Purpose**: Align project toolchains and add the API documentation dependency.

- [ ] T001 Update Java toolchain and CI versions to 21 in `pom.xml`, `build.gradle`, `.github/workflows/maven-build.yml`, and `.github/workflows/gradle-build.yml`.
- [ ] T002 Add `org.springdoc:springdoc-openapi-starter-webmvc-ui:3.2.1` to `pom.xml` and `build.gradle`.

## Phase 2: Foundational

**Purpose**: Confirm the shared build baseline before implementing user stories.

- [ ] T003 Run `./mvnw -B verify` and `./gradlew build` on Java 21 after T001-T002; resolve setup-related failures before continuing, recording any failure in the task review.

**Checkpoint**: Maven and Gradle both build on the constitution-mandated Java 21 toolchain.
No database migration, new authentication policy, or separate application project is
required.

## Phase 3: User Story 1 - Continue using the clinic UI (Priority: P1) 🎯 MVP

**Goal**: Preserve all existing MVC workflows while moving their business operations
behind services that can be shared with API controllers.

**Independent Test**: Run owner, pet, visit, veterinarian, and application integration
tests; verify HTML routes, view names, model behavior, redirects, validation messages, and
the legacy `GET /vets` JSON wrapper remain unchanged.

### Tests for User Story 1

- [ ] T004 [P] [US1] Add owner-service tests for last-name prefix search with one-based page handling, default page size 5, and owner lookup/save behavior in `src/test/java/org/springframework/samples/petclinic/service/OwnerServiceTests.java`.
- [ ] T005 [P] [US1] Add pet-service tests for required non-blank names up to 30 characters, required type on creation, non-future birth date, owner-scoped case-insensitive name uniqueness, and retaining type when omitted on update in `src/test/java/org/springframework/samples/petclinic/service/PetServiceTests.java`.
- [ ] T006 [P] [US1] Add visit and pet-type service tests for available-type listing, visit description required/non-blank up to 255 characters, and visit date strictly after today in `src/test/java/org/springframework/samples/petclinic/service/VisitServiceTests.java`.
- [ ] T007 [P] [US1] Add veterinarian-service tests for complete and paginated listing in `src/test/java/org/springframework/samples/petclinic/service/VetServiceTests.java`.

### Implementation for User Story 1

- [ ] T008 [P] [US1] Implement `OwnerService` for owner lookup, last-name prefix search, creation, and update using `OwnerRepository` in `src/main/java/org/springframework/samples/petclinic/owner/OwnerService.java`.
- [ ] T009 [P] [US1] Implement `VetService` for complete and paginated veterinarian listing using `VetRepository` in `src/main/java/org/springframework/samples/petclinic/vet/VetService.java`.
- [ ] T010 [US1] Implement `PetTypeService` for listing available pet types using `PetTypeRepository` in `src/main/java/org/springframework/samples/petclinic/owner/PetTypeService.java`.
- [ ] T011 [US1] Implement `PetService` for owner-scoped pet lookup, creation, update, and duplicate-name handling; enforce name non-blank/max 30, required creation type, birth date not after today, and case-insensitive uniqueness per owner in `src/main/java/org/springframework/samples/petclinic/owner/PetService.java`.
- [ ] T012 [US1] Implement `VisitService` for owner/pet-scoped visit booking; enforce description required/non-blank/max 255 and visit date after today in `src/main/java/org/springframework/samples/petclinic/owner/VisitService.java`.
- [ ] T013 [US1] Refactor `OwnerController` to use `OwnerService` while preserving `/owners` mappings, page-size-5 search behavior, view/model names, form errors, redirects, and flash messages in `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`.
- [ ] T014 [US1] Refactor `PetController` to use `PetService` and `PetTypeService` while preserving `/owners/{ownerId}/pets/**` mappings, binder protections, form views, duplicate-name feedback, and redirects in `src/main/java/org/springframework/samples/petclinic/owner/PetController.java`.
- [ ] T015 [US1] Refactor `VisitController` to use `VisitService` while preserving visit form binding, validation feedback, messages, and redirects in `src/main/java/org/springframework/samples/petclinic/owner/VisitController.java`.
- [ ] T016 [US1] Refactor `VetController` to use `VetService` while keeping `/vets.html` pagination and the exact `GET /vets` `Vets.vetList` JSON shape in `src/main/java/org/springframework/samples/petclinic/vet/VetController.java`.
- [ ] T017 [US1] Update MVC controller tests for service injection and assert existing route, view, model, form-validation, redirect, and legacy `/vets` behavior in `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java`, `src/test/java/org/springframework/samples/petclinic/owner/PetControllerTests.java`, `src/test/java/org/springframework/samples/petclinic/owner/VisitControllerTests.java`, and `src/test/java/org/springframework/samples/petclinic/vet/VetControllerTests.java`.
- [ ] T018 [US1] Run `./mvnw -Dtest='OwnerControllerTests,PetControllerTests,VisitControllerTests,VetControllerTests,PetClinicIntegrationTests' test` and `./gradlew test`; resolve regressions in `src/test/java/org/springframework/samples/petclinic/owner/` and `src/test/java/org/springframework/samples/petclinic/vet/`.

**Checkpoint**: Existing browser workflows and the veterinarian JSON compatibility
resource pass regression checks; services now own shared business behavior.

## Phase 4: User Story 2 - Use clinic capabilities from another frontend (Priority: P2)

**Goal**: Expose versioned REST operations for owners, pets, visits, pet types, and vets
using the shared services and stable documented contracts.

**Independent Test**: Use HTTP tests or an API client to complete every API operation;
verify success DTOs, validation errors, missing-resource errors, duplicate conflicts,
pagination, and runtime OpenAPI documentation.

### Tests for User Story 2

- [ ] T019 [P] [US2] Add MockMvc API tests for owner prefix search, pagination, get/create/update, DTO response shape, and request validation in `src/test/java/org/springframework/samples/petclinic/owner/api/OwnerApiControllerTests.java`.
- [ ] T020 [P] [US2] Add MockMvc API tests for pet create/update, owner scoping, required create type, optional update type, future birth dates, and duplicate-name conflicts in `src/test/java/org/springframework/samples/petclinic/owner/api/PetApiControllerTests.java`.
- [ ] T021 [P] [US2] Add MockMvc API tests for visit booking, pet-type lookup, missing owner/pet errors, and invalid page/size errors in `src/test/java/org/springframework/samples/petclinic/owner/api/ClinicApiControllerTests.java`.
- [ ] T022 [P] [US2] Add MockMvc API tests for versioned veterinarian listing and unchanged legacy `GET /vets` response in `src/test/java/org/springframework/samples/petclinic/vet/api/VetApiControllerTests.java`.

### Implementation for User Story 2

- [ ] T023 [P] [US2] Create record-based owner request/response DTOs; owner first/last name MUST be non-blank and max 30, address non-blank and max 255, city non-blank and max 80, telephone exactly 10 digits, and request DTOs MUST NOT accept IDs or nested pets in `src/main/java/org/springframework/samples/petclinic/owner/api/dto/OwnerRequest.java` and `src/main/java/org/springframework/samples/petclinic/owner/api/dto/OwnerResponse.java`.
- [ ] T024 [P] [US2] Create record-based pet request/response DTOs; pet name MUST be non-blank and max 30, birth date MUST NOT be after today, creation MUST require a pet type ID, update MAY omit type to retain the current type, and response DTOs MUST avoid owner back-references in `src/main/java/org/springframework/samples/petclinic/owner/api/dto/PetRequest.java`, `src/main/java/org/springframework/samples/petclinic/owner/api/dto/PetUpdateRequest.java`, and `src/main/java/org/springframework/samples/petclinic/owner/api/dto/PetResponse.java`.
- [ ] T025 [P] [US2] Create record-based visit and pet-type DTOs; visit description MUST be non-blank and max 255, visit date MUST be after today, and pet type responses MUST expose ID and name in `src/main/java/org/springframework/samples/petclinic/owner/api/dto/VisitRequest.java`, `src/main/java/org/springframework/samples/petclinic/owner/api/dto/VisitResponse.java`, and `src/main/java/org/springframework/samples/petclinic/owner/api/dto/PetTypeResponse.java`.
- [ ] T026 [P] [US2] Create record-based veterinarian and generic page response DTOs with veterinarian specialty names, one-based page, size, total elements, and total pages in `src/main/java/org/springframework/samples/petclinic/vet/api/dto/VetResponse.java` and `src/main/java/org/springframework/samples/petclinic/api/PageResponse.java`.
- [ ] T027 [US2] Implement API exception handling that returns RFC 9457 `application/problem+json` with stable `code`, field-level validation errors, safe details, 400 validation/input errors, 404 missing owner/pet errors, 409 duplicate-pet errors, and an `UNSUPPORTED_API_VERSION` error for unrecognized `/api/v{version}/**` paths in `src/main/java/org/springframework/samples/petclinic/api/ApiExceptionHandler.java` and `src/main/java/org/springframework/samples/petclinic/api/ApiProblem.java`.
- [ ] T028 [US2] Implement `/api/v1/owners` search/create and `/api/v1/owners/{ownerId}` get/update endpoints using `OwnerService`; return one-based pages, default size 5, maximum size 100, empty pages for no matches, 201 plus `Location` on creation, and 400 for invalid paging in `src/main/java/org/springframework/samples/petclinic/owner/api/OwnerApiController.java`.
- [ ] T029 [US2] Implement `/api/v1/owners/{ownerId}/pets` create and `/api/v1/owners/{ownerId}/pets/{petId}` update endpoints using `PetService`; return 201 plus `Location` on creation, 404 for foreign/missing pets, and 409 for duplicate names in `src/main/java/org/springframework/samples/petclinic/owner/api/PetApiController.java`.
- [ ] T030 [US2] Implement `/api/v1/owners/{ownerId}/pets/{petId}/visits` visit creation using `VisitService`; return 201 plus `Location` and reject invalid dates/descriptions or missing owner/pet with standardized errors in `src/main/java/org/springframework/samples/petclinic/owner/api/VisitApiController.java`.
- [ ] T031 [US2] Implement `GET /api/v1/pet-types` using `PetTypeService` and return record-based pet type DTOs in `src/main/java/org/springframework/samples/petclinic/owner/api/PetTypeApiController.java`.
- [ ] T032 [US2] Implement paginated `GET /api/v1/vets` using `VetService` and veterinarian DTOs while preserving `GET /vets` compatibility in `src/main/java/org/springframework/samples/petclinic/vet/api/VetApiController.java`.
- [ ] T033 [US2] Add Springdoc metadata and configure generated OpenAPI to document all `/api/v1` operations, success/error DTOs, URL versioning, and the legacy `/vets` compatibility exception in `src/main/java/org/springframework/samples/petclinic/api/OpenApiConfiguration.java`.
- [ ] T034 [US2] Update `specs/001-shared-services-rest-api/contracts/openapi.yaml` to match runtime paths, request/response DTOs, required field constraints, paging bounds, status codes, Problem Details, and the legacy veterinarian wrapper.
- [ ] T035 [US2] Add a runtime OpenAPI integration test for `/v3/api-docs` that asserts every documented versioned route and legacy `/vets` compatibility path is present in `src/test/java/org/springframework/samples/petclinic/api/OpenApiContractTests.java`.
- [ ] T036 [US2] Run focused API tests and verify valid and invalid requests against H2 through Maven and Gradle; resolve API contract and validation failures in `src/test/java/org/springframework/samples/petclinic/owner/api/`, `src/test/java/org/springframework/samples/petclinic/vet/api/`, `src/main/java/org/springframework/samples/petclinic/owner/api/`, and `src/main/java/org/springframework/samples/petclinic/vet/api/`.

**Checkpoint**: API clients can use all shared clinic capabilities with versioned,
validated, documented, and stable contracts.

## Phase 5: User Story 3 - Migrate frontend workflows incrementally (Priority: P3)

**Goal**: Prove that API and MVC workflows can coexist and observe the same business data,
without requiring a JavaScript frontend or removal of the UI in this feature.

**Independent Test**: Create or update records through the API and read them through MVC;
then perform a supported MVC change and read the updated record through the API. Confirm
unmigrated UI workflows and the legacy veterinarian resource remain available.

- [ ] T037 [US3] Add cross-interface integration tests for API-created owners/pets/visits appearing in MVC and MVC changes appearing in API responses in `src/test/java/org/springframework/samples/petclinic/PetClinicApiMigrationIntegrationTests.java`.
- [ ] T038 [US3] Verify the existing MVC routes and `/vets` compatibility resource remain available alongside all `/api/v1` routes in `src/test/java/org/springframework/samples/petclinic/PetClinicApiMigrationIntegrationTests.java`.

**Checkpoint**: The application supports a workflow-by-workflow frontend migration while
both interfaces operate on the same records and business rules.

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Complete documentation and execute whole-feature validation.

- [ ] T039 [P] Update `README.md` with Java 21, API base path, OpenAPI/Swagger UI locations, existing `/vets` compatibility, and the fact that the MVC UI remains supported.
- [ ] T040 Run the smoke checks in `specs/001-shared-services-rest-api/quickstart.md`, `./mvnw -B verify`, and `./gradlew build` on Java 21; resolve remaining failures in the affected feature files.

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No prerequisites. T001 and T002 touch shared build descriptors, so
  coordinate them rather than editing those files concurrently.
- **Foundational (Phase 2)**: Depends on Setup; its Java 21 build verification blocks story work.
- **User Story 1 (Phase 3)**: Depends on the Foundation; establishes shared services and
  proves existing UI behavior is preserved.
- **User Story 2 (Phase 4)**: Depends on US1 services and verified compatibility; provides
  the new API adapters and contract.
- **User Story 3 (Phase 5)**: Depends on US1 and US2; tests the migration/coexistence goal.
- **Polish (Phase 6)**: Depends on all desired stories.

### User Story Dependencies

- **US1 (P1)**: Can start after Phase 2. It establishes service behavior and preserves the
  legacy user interface.
- **US2 (P2)**: Depends on US1 because the API must use the shared services and not create
  parallel business logic.
- **US3 (P3)**: Depends on US2 because it verifies bidirectional use across the completed
  API and the existing MVC interface.

### Parallel Opportunities

- T001 and T002 are separate setup concerns but both edit the build descriptors; do not
  run them concurrently on those shared files.
- US1 service tests T004-T007 can be created in parallel; service implementation T008 and
  T009 can proceed independently, followed by pet-type/pet/visit services as their
  dependencies allow.
- US2 API test files T019-T022 can be written in parallel. DTO tasks T023-T026 affect
  separate files and can proceed in parallel after the API shapes are agreed.
- US2 endpoint tasks T028-T032 are parallelizable after DTOs and shared API error handling
  are ready, because they touch separate controllers.
- T039 can run independently of feature-code work; final verification T040 waits for all
  changes.

## Parallel Example: User Story 2

```text
After T023-T027 are complete:
Task: Implement owner API in src/main/java/org/springframework/samples/petclinic/owner/api/OwnerApiController.java
Task: Implement pet API in src/main/java/org/springframework/samples/petclinic/owner/api/PetApiController.java
Task: Implement visit API in src/main/java/org/springframework/samples/petclinic/owner/api/VisitApiController.java
Task: Implement pet type API in src/main/java/org/springframework/samples/petclinic/owner/api/PetTypeApiController.java
Task: Implement vet API in src/main/java/org/springframework/samples/petclinic/vet/api/VetApiController.java
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Java 21/OpenAPI setup and verify both build systems.
2. Extract shared services and refactor MVC controllers to delegate to them.
3. Run all current UI and legacy veterinarian regression tests.
4. Stop and validate that staff workflows remain unchanged before adding new API behavior.

### Incremental Delivery

1. Complete US1 to establish shared services and preserve current workflows.
2. Complete US2 to expose and document versioned API capabilities.
3. Complete US3 to verify API/MVC data consistency for gradual migration.
4. Run quickstart smoke checks plus both full build pipelines.

## Notes

- Every task uses the required checkbox, sequential ID, optional parallel marker, story
  label where applicable, and concrete file path.
- Tests are explicit because the spec requires regression tests and verifiable parity.
- No JavaScript frontend, data migration, delete operation, new authentication scheme, or
  CORS policy is included in this feature.
