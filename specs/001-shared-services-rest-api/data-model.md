# Data Model: Shared Services and REST API

The API uses dedicated DTOs; persistent JPA entities remain internal. Entity identifiers
are server-managed. The relations and field rules below describe the current clinic domain
and its API representation, not a database migration.

## Owner

Represents a clinic client.

| Field | Type | Constraints |
|---|---|---|
| id | integer | Server-assigned; positive |
| firstName | string | Required; non-blank; max 30 characters |
| lastName | string | Required; non-blank; max 30 characters |
| address | string | Required; non-blank; max 255 characters |
| city | string | Required; non-blank; max 80 characters |
| telephone | string | Required; exactly 10 digits |
| pets | list of Pet | Zero or more pets owned by this owner |

**Relationships**: An owner has zero or more pets. An API pet mutation is always scoped
under its owner; clients cannot transfer a pet by changing an owner identifier in a
request body.

**Search**: Prefix match on last name. API pagination is one-based; default page is 1,
default size is 5, maximum size is 100. An empty result is a successful empty page.

## Pet

Represents an animal registered to an owner.

| Field | Type | Constraints |
|---|---|---|
| id | integer | Server-assigned; positive |
| name | string | Required; non-blank; max 30 characters |
| birthDate | date | Required; cannot be after today |
| type | PetType | Required on creation |
| visits | list of Visit | Zero or more visits, ordered by visit date |

**Relationships and uniqueness**: A pet belongs to exactly one owner; its name must be
unique within that owner, case-insensitively. A pet update must identify a pet belonging
to the path owner. Creation requires a pet type. On update, `petTypeId` is optional; when
omitted, the existing pet type is retained, matching current MVC form binding.

## Visit

Represents a visit booked for a pet.

| Field | Type | Constraints |
|---|---|---|
| id | integer | Server-assigned; positive |
| date | date | Required; must be after today |
| description | string | Required; non-blank; max 255 characters |

**Relationships**: A visit belongs to one pet and is read through that pet and its owner.
The API books visits using the nested owner and pet route.

## PetType

Represents a configured category available when registering a pet.

| Field | Type | Constraints |
|---|---|---|
| id | integer | Server-assigned; positive |
| name | string | Required |

The API exposes available pet types so API consumers can select valid values for pet
creation and editing.

## Veterinarian

Represents a clinic professional.

| Field | Type | Constraints |
|---|---|---|
| id | integer | Server-assigned; positive |
| firstName | string | Existing domain validation applies |
| lastName | string | Existing domain validation applies |
| specialties | list of strings | Zero or more specialty names |

The versioned API uses an explicit response DTO. The legacy `GET /vets` representation
continues to expose the current `Vets` wrapper with `vetList`.

## API DTOs

- `OwnerRequest`: first name, last name, address, city, and telephone. Does not accept an
  ID or nested pets.
- `OwnerResponse`: owner fields and nested pet responses. Pet responses include
  type and visit summaries; they do not contain back-references to owner.
- `PetRequest`: name, birth date, and pet type ID.
- `PetUpdateRequest`: name and birth date, with an optional pet type ID; omission retains
  the current type.
- `PetResponse`: pet fields, type summary, and visit summaries.
- `VisitRequest`: date and description.
- `VisitResponse`: visit ID, date, and description.
- `PetTypeResponse`: pet type ID and name.
- `VetResponse`: veterinarian ID, name fields, and specialty names.
- `PageResponse<T>`: items, one-based page number, requested page size, total item count,
  and total page count.
- `ApiProblem`: RFC 9457 fields plus stable application code and optional field errors.

## State and Conflict Handling

- Owner, pet, and visit IDs are assigned by the system and cannot be supplied or changed
  through API request DTOs.
- A missing owner or a pet not owned by the path owner returns a not-found API problem.
- A duplicate pet name for the same owner returns a conflict API problem and changes no
  records.
- Invalid request fields return a validation problem and do not persist a partial change.
- A successful write is immediately visible to reads from either MVC or REST.
