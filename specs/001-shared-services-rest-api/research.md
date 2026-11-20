# Research: Shared Services and REST API

## Decision: Extract domain services and retain MVC as a presentation adapter

**Decision**: Move business operations currently implemented in owner, pet, visit, and
veterinarian controllers behind feature-centered application services. Both MVC and REST
controllers call those services. Keep view names, model attributes, form binding, redirects,
and flash messages in the MVC controllers.

**Rationale**: Current MVC controllers call repositories directly. The shared services
provide one place for use-case behavior and ensure that API and browser operations apply
the same rules. This preserves current MVC responsibilities and avoids a parallel business
logic implementation.

**Alternatives considered**:

- Keep repository access in each controller: rejected because API and MVC behavior could
  diverge.
- Replace MVC with REST and a JavaScript client in the same change: rejected because the
  feature requires unchanged workflows and gradual migration.
- Add a separate backend deployment: rejected because it adds deployment and data
  consistency complexity without a stated need.

**Repository evidence**: `owner/OwnerController`, `owner/PetController`, and
`owner/VisitController` directly invoke repositories. `vet/VetController` directly invokes
`VetRepository`. Existing MVC tests exercise these routes and views.

## Decision: Preserve existing workflows and the veterinarian JSON resource

**Decision**: Keep all current HTML mappings and the exact behavior of `GET /vets`, whose
JSON body wraps the vet list under `vetList`. Add new REST operations at `/api/v1/**`.

**Rationale**: The compatibility requirement includes current application functions, not
only the HTML UI. The `/vets` payload is already exercised by a controller test and could
be consumed independently of the HTML page at `/vets.html`.

**Alternatives considered**:

- Move or rename `/vets` to `/api/v1/vets`: rejected because it would break existing
  consumers.
- Change the legacy wrapper to a page response: rejected because it changes the existing
  JSON shape.

**Repository evidence**: `VetController.showResourcesVetList()` maps `GET /vets` and
returns `Vets`; `VetControllerTests.showResourcesVetList()` asserts JSON and
`$.vetList[0].id`.

## Decision: Use Springdoc OpenAPI 3 for the Spring MVC application

**Decision**: Add `org.springdoc:springdoc-openapi-starter-webmvc-ui:3.2.1` consistently
to Maven and Gradle. Generate the runtime OpenAPI document and Swagger UI; maintain a
reviewed `contracts/openapi.yaml` that describes new versioned routes and the legacy vet
resource.

**Rationale**: Springdoc's official getting-started guide identifies this dependency and
documents `/v3/api-docs`, `/v3/api-docs.yaml`, and the Swagger UI path. Its current project
README advertises Spring Boot 4.x support and Java 17+, which covers the repository's Boot
4.1.0 and Java 21 target. Pinning the same version in both build systems avoids divergent
contracts.

**Alternatives considered**:

- Maintain OpenAPI YAML only by hand: rejected as the only source because it can drift from
  Spring controller behavior.
- Omit the OpenAPI runtime: rejected because the constitution requires API documentation.

**Sources**:

- Springdoc getting started: <https://springdoc.org/getting-started.html>
- Springdoc project and compatibility badges: <https://github.com/springdoc/springdoc-openapi>

## Decision: Use explicit API DTOs and Problem Details

**Decision**: Define record-based request and response DTOs, with nested response shapes
for owners, pets, pet types, visits, and veterinarians. Never expose JPA entities directly.
Return RFC 9457 Problem Details with a stable application error code and structured field
errors.

**Rationale**: `Owner` contains pets and `Pet` contains visits and a pet type. Serializing
entities would couple public JSON to persistence details and risks recursive or unstable
payloads. Problem Details gives API clients a predictable error envelope without leaking
exception internals.

**Alternatives considered**:

- Return entities directly: rejected for persistence coupling and serialization risks.
- Return unstructured text for failures: rejected because consumers need a stable,
  machine-readable error shape.

## Decision: Preserve validation and search semantics at the service boundary

**Decision**: Keep existing owner prefix search, five-item default page, owner and pet
validation, per-owner duplicate pet-name protection, and visits only after the current
date. Make the API's search page one-based; allow `size` up to 100. Return an empty page
for no matches. A new pet requires a type; an update may omit its type and retain the
existing value, matching the MVC form's binding of an existing pet before validation.
Invalid page or size values return a standardized 400 API error; the HTML UI retains its
current redirect behavior for invalid page numbers.

**Rationale**: These choices keep existing business semantics and UI defaults intact while
giving API clients a conventional paged result. The bounded size prevents accidental
unbounded result requests.

**Repository evidence**: `OwnerController` normalizes prefix search and uses page size 5;
`PetValidator` validates pet names, type on creation, and birth date; `PetController`
checks duplicate names; `VisitController` rejects today or past dates.

## Decision: Align build and CI toolchains to Java 21

**Decision**: As a feature prerequisite, update Java version settings in `pom.xml`,
`build.gradle`, `.github/workflows/maven-build.yml`, and
`.github/workflows/gradle-build.yml` from 17 to 21.

**Rationale**: The project constitution makes Java 21 LTS mandatory, while all four
currently inspected settings use 17. Leaving the mismatch would make the feature violate
project governance and could allow CI to pass on a non-conforming runtime.

**Alternatives considered**:

- Leave the existing build configuration unchanged: rejected because it conflicts with the
  constitution.
- Change only one build tool: rejected because the repository maintains and runs both Maven
  and Gradle builds in CI.

## Decision: Keep migration same-origin and defer frontend hosting concerns

**Decision**: Do not add a new CORS policy or frontend application in this feature. The
REST API follows existing application access/configuration policy and is usable through
same-origin clients.

**Rationale**: The spec requires APIs to enable a later JavaScript frontend but does not
require cross-origin hosting or a frontend implementation. A permissive CORS policy would
expand access without a confirmed deployment need.

## Outstanding quality targets

The feature specification does not define latency, throughput, availability, or concurrent
edit targets. No new operational target is introduced; retain existing Actuator/Micrometer
behavior and establish measurable targets only if deployment requirements call for them.
