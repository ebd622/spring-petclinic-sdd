# Spring PetClinic — C4 System Context

```mermaid
C4Context
  title Spring PetClinic — System Context

  Person(staff, "Clinic staff", "Looks up owners, pets, and veterinarians; records visits.")
  System(petclinic, "Spring PetClinic", "Web application for managing clinic owners, pets, visits, and veterinarians.")

  Rel(staff, petclinic, "Uses")
```
