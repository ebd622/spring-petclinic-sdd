# Spring PetClinic Sample Application 

## Goal of the refactoring
Evolve the Spring Boot application from a server-side MVC architecture to an API-driven architecture.

### Constitution
```aiignore
/speckit-constitution
/skill:speckit-constitution
```
```aiignore
/skill:speckit-constitution
- Java 21 LTS is mandatory.
- Follow REST API best practices and OpenAPI 3 documentation.
- Use feature-based package structure.
- Use Java Records for DTOs where appropriate.
- Implement input validation using Jakarta Validation.
- Use Spring Boot Actuator and Micrometer for observability.
- Use secure coding practices, externalized configuration, and no hard-coded secrets.
- Prefer clean code, SOLID principles, and maintainable architecture.
- Require API versioning and standardized error responses.
```

### Specify
```aiignore
/skill:speckit-specify 
Refactor the Spring Boot application to support both the existing MVC UI and new REST APIs. 
Keep all current functionality and user workflows unchanged.

Introduce a shared service layer used by both MVC and API controllers, exposing the same business capabilities through REST endpoints. 
This will enable a gradual migration to a React, Angular, or other JavaScript frontend.

The long-term goal is to replace the MVC UI entirely and move to an API-driven architecture without a big-bang rewrite.
```

### Clarify
```aiignore
/skill:speckit-clarify
```
Now it is a good time to read and check your specification.

### Plan
```aiignore
/skill:speckit-plan
```

### Tasks
```
/skill:speckit-tasks
```
Run tasks command without any input
