# Quickstart: Shared Services and REST API

## Prerequisites

- Java 21 LTS installed and selected.
- Docker available for Testcontainers-based MySQL/PostgreSQL integration tests.
- Dependencies resolvable from the configured Maven and Gradle repositories.

## Build and regression validation

Run both supported build pipelines because the repository maintains Maven and Gradle
builds:

```bash
./mvnw -B verify
./gradlew build
```

Expected result: both builds pass on Java 21, existing MVC and `/vets` compatibility
tests pass, and new service and API contract tests pass.

For focused development, run the related test classes through the relevant build tool:

```bash
./mvnw -Dtest='*Owner*Tests,*Pet*Tests,*Visit*Tests,*Vet*Tests,*Api*Tests' test
./gradlew test
```

Run the database integration profiles when changing persistence behavior; Docker must be
available for container-backed tests.

## API smoke checks

Start the application with the default H2 profile:

```bash
./mvnw spring-boot:run
```

In another terminal, confirm that:

1. `GET /api/v1/pet-types` returns the configured pet types.
2. `GET /api/v1/owners?lastName=&page=1&size=5` returns a page response.
3. `GET /api/v1/owners/1` returns the owner and nested pets without recursive JSON.
4. `GET /api/v1/vets` returns veterinarian DTOs.
5. `GET /vets` retains the legacy `vetList` response shape.
6. `GET /v3/api-docs` returns the OpenAPI document and lists all versioned routes.
7. An invalid request returns `application/problem+json` with a stable error code and
   field-level validation details, without internal exception information.

The checked-in interface contract is [contracts/openapi.yaml](contracts/openapi.yaml);
domain fields and validation rules are described in [data-model.md](data-model.md).
