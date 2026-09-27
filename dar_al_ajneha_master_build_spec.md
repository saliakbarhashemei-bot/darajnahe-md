
# Dar Al-Ajneha — Master Build Specification

## Purpose

This file is the implementation specification for the Dar Al-Ajneha admin system.

It combines the approved System Map and Relationship Key with the engineering rules required to build the system safely.

**Source of truth:** the System Map and its 36 Relationship Key cards below.

**Core rule:** do not invent, remove, rename, merge, or silently reinterpret Domains, Categories, Subcategories, relationships, statuses, roles, or permissions.

---

# 1. Product Scope

Dar Al-Ajneha is a professional web-based administration panel for a publishing company.

The system manages:

- Users & Access
- Books
- Contributors
- Media
- Announcements
- Cross-domain relationships

This specification is for a web admin system. There is no mobile-app requirement.

---

# 2. Required Stack

## Frontend

- React
- TypeScript
- Component-based architecture
- REST API communication

## Backend

- PHP
- Laravel
- Eloquent ORM
- Form Requests / backend validation
- Policies / authorization
- API Resources where useful

## Database

- PostgreSQL

## Storage

Use Laravel's storage abstraction so local development and S3-compatible/object storage can both be supported.

---

# 3. Non-Negotiable Agent Rules

1. Inspect the existing repository before changing it.
2. Preserve working code.
3. Do not rewrite unrelated modules.
4. Backend authorization is authoritative.
5. Validate all client input on the backend.
6. Never trust frontend permission checks as security.
7. Do not invent business requirements.
8. Do not add a mobile app.
9. Do not introduce microservices unless explicitly requested.
10. Do not add unnecessary packages or architecture layers.
11. Run relevant tests after changes.
12. Never claim completion without verification.
13. If implementation requires changing the approved System Map, stop at that boundary and report the conflict instead of silently changing it.

---

# 4. Important Source-Map Exceptions

The current Relationship Key contains two references that are **not present as normal items in the corresponding Domain tree**:

- `CONTRIBUTORS → Contributions → Books`
- `ANNOUNCEMENTS → Content → Image / File`

They must remain in the Relationship Key because they are part of the approved 36-card map.

However:

- do not silently add `Contributions` to the CONTRIBUTORS Domain tree
- do not silently add `Image / File` to the ANNOUNCEMENTS Content tree
- do not invent fields or workflows for either
- treat them as relationship-only references until explicitly revised

This is intentional change-control behavior.

---

# 5. Architecture

Use a **modular monolith**:

```text
React + TypeScript
        |
        | REST API
        v
Laravel / PHP
        |
        +-- USER & ACCESS
        +-- BOOKS
        +-- CONTRIBUTORS
        +-- MEDIA
        +-- ANNOUNCEMENTS
        |
        v
PostgreSQL

Laravel Storage Abstraction
        |
        v
Local Storage / S3-compatible Storage
```

Do not split the first version into microservices.

---

# 6. Backend Structure

Keep Laravel conventions while maintaining clear domain boundaries.

Conceptually:

```text
app/
├── Models/
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   └── Resources/
├── Policies/
├── Services/
└── Support/

database/
├── migrations/
├── seeders/
└── factories/

routes/
└── api.php

tests/
├── Feature/
└── Unit/
```

Do not create abstractions merely for the sake of having abstractions.

---

# 7. Database Rules

Use PostgreSQL relational modeling.

Required rules:

- primary keys
- foreign keys
- appropriate indexes
- unique constraints where needed
- timestamps
- safe archive/soft-delete behavior where appropriate
- normalized relationships
- no comma-separated relational IDs
- no duplicated person tables for contributor roles
- no JSON used as a replacement for ordinary relational modeling

---

# 8. Core Domain Model

## Users

Administrative users.

Must support:

- authentication identity
- active/disabled state
- roles
- timestamps
- safe password handling

## Roles

Authorization roles.

## Permissions

Granular authorization permissions.

Roles and permissions should use a many-to-many relationship.

## Books

Must support:

- title
- ISBN
- description
- page count
- language
- publication date
- publisher
- edition
- status
- contributor relationships
- classification
- media relationships

## Contributors / People

One reusable person entity.

Do NOT create separate person systems for:

- Author
- Translator
- Editor
- Illustrator

A person can have roles in book relationships.

## Book-Person relationship

Must be capable of representing:

- book
- person
- role

A book can have multiple contributors.
A contributor can participate in multiple books.

## Media

Stores file metadata and storage references.

Do not treat the physical file itself as the database record.

## Announcements

Must support:

- title
- short description
- content
- type
- status
- timestamps

---

# 9. Controlled Values

## Book status

- Draft
- Review
- Scheduled
- Published
- Archived

## Announcement status

- Draft
- Review
- Scheduled
- Published
- Archived

## Contributor roles

- Author
- Translator
- Editor
- Illustrator

## Announcement types

- New Book
- Reprint
- New Edition
- News
- Event
- Discount
- Other

## Media types

- Book Cover
- Book Image
- Person Photo
- Announcement Image
- Publisher Logo
- Document

Do not add values without explicit approval.

---

# 10. API Rules

Use a versioned REST API, preferably:

`/api/v1`

Representative resources:

```text
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/me

GET    /api/v1/books
POST   /api/v1/books
GET    /api/v1/books/{book}
PUT    /api/v1/books/{book}
DELETE /api/v1/books/{book}

GET    /api/v1/contributors
POST   /api/v1/contributors
GET    /api/v1/contributors/{contributor}
PUT    /api/v1/contributors/{contributor}
DELETE /api/v1/contributors/{contributor}

GET    /api/v1/media
POST   /api/v1/media
GET    /api/v1/media/{media}
PUT    /api/v1/media/{media}
DELETE /api/v1/media/{media}

GET    /api/v1/announcements
POST   /api/v1/announcements
GET    /api/v1/announcements/{announcement}
PUT    /api/v1/announcements/{announcement}
DELETE /api/v1/announcements/{announcement}
```

These are resource examples, not permission to invent new business entities.

API requirements:

- consistent JSON responses
- field-level validation errors
- correct HTTP status codes
- safe error messages
- no SQL/stack trace leakage
- pagination for collection endpoints
- validated filtering/search parameters
- avoid N+1 queries

---

# 11. Authentication and Authorization

Authentication:

- login
- logout
- session/authenticated state
- password management

Passwords:

- hash securely
- never store plaintext
- never return passwords
- never log passwords

Authorization must be enforced server-side.

Permission-controlled operations include:

- Create/Edit/Archive/Delete Book
- Create/Edit/Archive/Delete Person
- Upload/Edit/Replace/Archive/Delete Media
- Create/Edit/Archive/Delete Announcement
- Create/Edit/Disable User

The Relationship Key is the source of truth for these mappings.

---

# 12. Media Rules

Media must support:

- upload
- view
- edit metadata
- replace
- archive
- delete
- library browsing
- type classification

Validate:

- MIME type
- size
- allowed media type
- filename handling
- dimensions where relevant

Never trust the filename extension alone.

Replacing media must not accidentally destroy a still-referenced asset.

---

# 13. Frontend Structure

Conceptually:

```text
src/
├── app/
├── components/
├── layouts/
├── pages/
│   ├── auth/
│   ├── books/
│   ├── contributors/
│   ├── media/
│   ├── announcements/
│   └── users/
├── features/
│   ├── books/
│   ├── contributors/
│   ├── media/
│   ├── announcements/
│   └── users/
├── services/
├── hooks/
├── types/
├── utils/
└── routes/
```

The frontend should mirror the domain boundaries conceptually.

Required UI states:

- loading
- empty
- success
- validation error
- unauthorized
- server error
- not found

Destructive actions require confirmation.

Archive and Delete must be clearly distinguishable.

---

# 14. Books UI

Support:

- list
- view
- create
- edit
- archive
- delete

Book detail should expose relevant:

- Book Information
- People & Relations
- Publishing
- Classification
- Media

Do not duplicate contributor records inside books.

---

# 15. Contributors UI

Support:

- list
- view
- create person
- edit
- archive
- delete

Contributor detail should expose:

- person information
- roles
- related books where applicable

A person must be reusable across books.

---

# 16. Media UI

Provide a media library supporting:

- all media
- images
- documents
- other files
- media type
- metadata
- storage information

Media should be reusable by the relevant business domains.

---

# 17. Announcements UI

Support:

- list
- view
- create
- edit
- archive
- delete

Use only the approved types and statuses.

---

# 18. Validation and Security

Validate both frontend and backend.

Backend validation is mandatory for:

- required fields
- types
- IDs
- relationships
- status values
- announcement types
- contributor roles
- ISBN where applicable
- non-negative page counts
- media files

Security baseline:

- secure password hashing
- authorization policies
- input validation
- mass-assignment protection
- safe file uploads
- parameterized database queries / ORM
- XSS-safe rendering
- rate limiting for sensitive endpoints
- secure environment secrets
- no secrets in source control

---

# 19. Search and Lists

Use server-side pagination.

Use validated search/filter parameters.

Avoid loading unlimited data into the browser.

Avoid N+1 queries.

Use eager loading when appropriate.

---

# 20. Archive/Delete Rules

Archive and Delete are not interchangeable.

Do not blindly cascade-delete important business data.

Before defining cascade behavior, inspect the business meaning of the relationship.

Prefer safe archive/soft-delete behavior where the Domain requires Archive.

---

# 21. Auditability

The system should be able to record important administrative activity with:

- actor/user
- action
- entity type
- entity ID
- timestamp
- optional relevant metadata

Never log credentials.

Audit/activity is cross-cutting. Do not turn it into a new business Domain without explicit approval.

---

# 22. Dashboard

Dashboard is a UI/view, not a new Domain.

Do not create a DASHBOARD Domain.

It may aggregate information from existing Domains.

---

# 23. Build Order

Do not build the whole system in one uncontrolled pass.

## Phase 0 — Foundation

- inspect repository
- confirm framework versions
- configure environment
- database connection
- API base
- lint/formatting
- testing
- baseline

Do not implement unrelated business features yet.

## Phase 1 — USER & ACCESS

Implement:

- authentication
- users
- roles
- permissions
- authorization
- user management

Verify authorization before moving on.

## Phase 2 — CONTRIBUTORS

Implement:

- person management
- roles
- person information
- book/contributor relationship foundation

## Phase 3 — BOOKS

Implement:

- book management
- book information
- publishing
- classification
- contributor relationships

## Phase 4 — MEDIA

Implement:

- media management
- library
- media types
- metadata
- storage
- Book ↔ Media
- Contributor ↔ Media
- Announcement ↔ Media

## Phase 5 — ANNOUNCEMENTS

Implement:

- management
- content
- types
- statuses
- documented Book business relationships

## Phase 6 — Integration

Verify all 36 relationship cards and all authorization mappings.

## Phase 7 — Hardening

Run:

- unit tests
- feature tests
- frontend tests
- authorization tests
- validation tests
- security review
- migration review
- performance review

---

# 24. Coding-Agent Workflow

For every task:

### 1. Inspect
Read relevant existing code first.

### 2. Map
Identify the affected:

- Domain
- Category
- Subcategory
- Relationship Package
- database entities
- API endpoints
- frontend screens/components

### 3. Plan
Create a short implementation plan.

### 4. Implement
Make the smallest coherent change.

### 5. Test
Run relevant tests.

### 6. Verify
Check database, API, authorization, UI, and relationships.

### 7. Report
Report:

- files changed
- implemented behavior
- tests run
- test results
- known limitations
- items requiring approval

Never report success without verification.

---

# 25. Definition of Done

A phase is complete only when:

- database schema works
- migrations work from a clean database
- seeders work
- backend API works
- validation works
- authorization works
- frontend works
- loading/empty/error states work
- relationships work
- relevant tests pass
- existing functionality remains intact
- implementation matches this specification

---

# 26. Change Control

Explicit approval is required before changing:

- Domain names
- Categories
- Subcategories
- Relationship Key
- statuses
- contributor roles
- announcement types
- permission model
- authentication model
- database architecture
- technology stack

Do not silently “improve” the business model.

---

# 27. Final Invariant

The implementation must follow:

```text
SYSTEM MAP
    ↓
DOMAIN
    ↓
CATEGORY
    ↓
SUBCATEGORY
    ↓
RELATIONSHIP
    ↓
DATABASE MODEL
    ↓
API
    ↓
FRONTEND
```

The code must adapt to the approved business model, not replace it.

---

# 28. Final Verification Checklist

- [ ] 5 Domains exist
- [ ] all approved Categories exist
- [ ] all approved Subcategories exist
- [ ] all 36 Relationship Key cards are accounted for
- [ ] no undocumented relationship was invented
- [ ] relationship-only references remain traceable
- [ ] authentication works
- [ ] authorization works
- [ ] users work
- [ ] roles work
- [ ] permissions work
- [ ] books work
- [ ] contributors work
- [ ] contributor roles work
- [ ] book/contributor relationships work
- [ ] media works
- [ ] media types work
- [ ] file storage works
- [ ] announcements work
- [ ] announcement types work
- [ ] announcement statuses work
- [ ] Book/Announcement business mappings work
- [ ] permission mappings work
- [ ] backend validation works
- [ ] frontend validation feedback works
- [ ] loading states work
- [ ] empty states work
- [ ] error states work
- [ ] archive behavior works
- [ ] delete behavior is safe
- [ ] API responses are consistent
- [ ] clean migrations work
- [ ] seeders work
- [ ] tests pass
- [ ] no secrets are committed
- [ ] no mobile app has been introduced
- [ ] no unnecessary microservices have been introduced

---

# END OF BUILD SPECIFICATION

---

# APPENDIX A — APPROVED SYSTEM MAP

The following is the exact project System Map provided as the source material. It is preserved here so the coding agent has the domain map and implementation instructions in one file.


# Dar Al-Ajneha — System Map

> Source of truth: the current System Map / Relationship Key diagram provided for this project.
>
> The document preserves the five Domains, their Categories/Subcategories, the cross-domain relationship packages, and all **36 relationship cards** shown in the Relationship Key.

---

# 1. System Overview

The system is organized into five primary Domains:

1. **USER & ACCESS**
2. **BOOKS**
3. **CONTRIBUTORS**
4. **MEDIA**
5. **ANNOUNCEMENTS**

The Relationship Map is organized by **Domain → Category → Subcategory** on both sides.

A relationship package groups multiple Subcategory-to-Subcategory relationships when they belong to the same **Category ↔ Category** relationship.

---

# 2. Domains

## 01. USER & ACCESS

### Authentication
- Login
- Logout
- Session
- Password

### Authorization
- Roles
- Permissions
- Access Rules

### User Management
- Create User
- Edit User
- Disable User
- Delete / Archive User

---

## 02. BOOKS

### Book Management
- Create Book
- View Book
- Edit Book
- Archive Book
- Delete Book

### Book Information
- Title
- ISBN
- Description
- Page Count
- Language
- Publication Date

### People & Relations
- Author
- Translator
- Editor
- Illustrator

### Publishing
- Publisher
- Edition
- Publication Date
- Status
  - Draft
  - Review
  - Scheduled
  - Published
  - Archived

### Classification
- Category
- Genre
- Tags

### Media
- Cover
- Images
- Files

---

## 03. CONTRIBUTORS

### Person Management
- Create Person
- View Person
- Edit Person
- Archive Person
- Delete Person

### Roles
- Author
- Translator
- Editor
- Illustrator

### Person Information
- Name
- Biography
- Photo
- Contact Information

---

## 04. MEDIA

### Media Management
- Upload Media
- View Media
- Edit Media
- Replace Media
- Archive Media
- Delete Media

### Media Library
- All Media
- Images
- Documents
- Other Files

### Media Types
- Book Cover
- Book Image
- Person Photo
- Announcement Image
- Publisher Logo
- Document

### Media Information
- File Name
- File Type
- File Size
- Dimensions
- Alt Text
- Caption
- Description

### Storage
- File Storage
- File Path / URL
- Storage Provider

### Relations
- Books
- Contributors
- Announcements

---

## 05. ANNOUNCEMENTS

### Announcement Management
- Create Announcement
- View Announcement
- Edit Announcement
- Archive Announcement
- Delete Announcement

### Content
- Title
- Short Description
- Content

### Type
- New Book
- Reprint
- New Edition
- News
- Event
- Discount
- Other

### Status
- Draft
- Review
- Scheduled
- Published
- Archived

> **Relationship Key note:** the current diagram also contains an `Image / File ↔ Announcement Image` relationship under the ANNOUNCEMENTS ↔ MEDIA package. This relationship is preserved below exactly as represented in the Relationship Key.

---

# 3. Relationship Key

## Package 01 — BOOKS ↔ CONTRIBUTORS

### People & Relations ↔ Roles

| Card | BOOKS | CONTRIBUTORS |
|---:|---|---|
| 01 | Author | Author |
| 02 | Translator | Translator |
| 03 | Editor | Editor |
| 04 | Illustrator | Illustrator |

Full path:

`BOOKS → People & Relations → Author ↔ CONTRIBUTORS → Roles → Author`

`BOOKS → People & Relations → Translator ↔ CONTRIBUTORS → Roles → Translator`

`BOOKS → People & Relations → Editor ↔ CONTRIBUTORS → Roles → Editor`

`BOOKS → People & Relations → Illustrator ↔ CONTRIBUTORS → Roles → Illustrator`

---

## Package 02 — BOOKS ↔ CONTRIBUTORS

### Book Management ↔ Contributions

| Card | BOOKS | CONTRIBUTORS |
|---:|---|---|
| 05 | View Book | Books |

Full path:

`BOOKS → Book Management → View Book ↔ CONTRIBUTORS → Contributions → Books`

---

## Package 03 — ANNOUNCEMENTS ↔ MEDIA

### Content ↔ Media Types

| Card | ANNOUNCEMENTS | MEDIA |
|---:|---|---|
| 06 | Image / File | Announcement Image |

Full path:

`ANNOUNCEMENTS → Content → Image / File ↔ MEDIA → Media Types → Announcement Image`

---

## Package 04 — USER & ACCESS ↔ CONTRIBUTORS

### Authorization ↔ Person Management

| Card | USER & ACCESS | CONTRIBUTORS |
|---:|---|---|
| 07 | Permissions | Create Person |
| 08 | Permissions | Edit Person |
| 09 | Permissions | Archive Person |
| 10 | Permissions | Delete Person |

Full paths:

`USER & ACCESS → Authorization → Permissions → CONTRIBUTORS → Person Management → Create Person`

`USER & ACCESS → Authorization → Permissions → CONTRIBUTORS → Person Management → Edit Person`

`USER & ACCESS → Authorization → Permissions → CONTRIBUTORS → Person Management → Archive Person`

`USER & ACCESS → Authorization → Permissions → CONTRIBUTORS → Person Management → Delete Person`

---

## Package 05 — BOOKS ↔ MEDIA

### Media ↔ Media Types

| Card | BOOKS | MEDIA |
|---:|---|---|
| 11 | Cover | Book Cover |
| 12 | Images | Book Image |
| 13 | Files | Document |

Full paths:

`BOOKS → Media → Cover ↔ MEDIA → Media Types → Book Cover`

`BOOKS → Media → Images ↔ MEDIA → Media Types → Book Image`

`BOOKS → Media → Files ↔ MEDIA → Media Types → Document`

---

## Package 06 — CONTRIBUTORS ↔ MEDIA

### Person Information ↔ Media Types

| Card | CONTRIBUTORS | MEDIA |
|---:|---|---|
| 14 | Photo | Person Photo |

Full path:

`CONTRIBUTORS → Person Information → Photo ↔ MEDIA → Media Types → Person Photo`

---

## Package 07 — USER & ACCESS ↔ MEDIA

### Authorization ↔ Media Management

| Card | USER & ACCESS | MEDIA |
|---:|---|---|
| 15 | Permissions | Upload Media |
| 16 | Permissions | Edit Media |
| 17 | Permissions | Replace Media |
| 18 | Permissions | Archive Media |
| 19 | Permissions | Delete Media |

Full paths:

`USER & ACCESS → Authorization → Permissions → MEDIA → Media Management → Upload Media`

`USER & ACCESS → Authorization → Permissions → MEDIA → Media Management → Edit Media`

`USER & ACCESS → Authorization → Permissions → MEDIA → Media Management → Replace Media`

`USER & ACCESS → Authorization → Permissions → MEDIA → Media Management → Archive Media`

`USER & ACCESS → Authorization → Permissions → MEDIA → Media Management → Delete Media`

---

## Package 08 — USER & ACCESS ↔ BOOKS

### Authorization ↔ Book Management

| Card | USER & ACCESS | BOOKS |
|---:|---|---|
| 20 | Permissions | Create Book |
| 21 | Permissions | Edit Book |
| 22 | Permissions | Archive Book |
| 23 | Permissions | Delete Book |

Full paths:

`USER & ACCESS → Authorization → Permissions → BOOKS → Book Management → Create Book`

`USER & ACCESS → Authorization → Permissions → BOOKS → Book Management → Edit Book`

`USER & ACCESS → Authorization → Permissions → BOOKS → Book Management → Archive Book`

`USER & ACCESS → Authorization → Permissions → BOOKS → Book Management → Delete Book`

---

## Package 09 — ANNOUNCEMENTS ↔ BOOKS

### Type ↔ Book Management / Publishing

| Card | ANNOUNCEMENTS | BOOKS |
|---:|---|---|
| 24 | New Book | Create Book |
| 25 | Reprint | Publication Date |
| 26 | Reprint | Status |
| 27 | New Edition | Edition |
| 28 | New Edition | Publication Date |

Full paths:

`ANNOUNCEMENTS → Type → New Book ↔ BOOKS → Book Management → Create Book`

`ANNOUNCEMENTS → Type → Reprint ↔ BOOKS → Publishing → Publication Date`

`ANNOUNCEMENTS → Type → Reprint ↔ BOOKS → Publishing → Status`

`ANNOUNCEMENTS → Type → New Edition ↔ BOOKS → Publishing → Edition`

`ANNOUNCEMENTS → Type → New Edition ↔ BOOKS → Publishing → Publication Date`

---

## Package 10 — ANNOUNCEMENTS ↔ MEDIA

### Announcement Management ↔ Media Types

| Card | ANNOUNCEMENTS | MEDIA |
|---:|---|---|
| 29 | Create / Edit Announcement | Announcement Image |

Full path:

`ANNOUNCEMENTS → Announcement Management → Create / Edit Announcement ↔ MEDIA → Media Types → Announcement Image`

---

## Package 11 — USER & ACCESS ↔ ANNOUNCEMENTS

### Authorization ↔ Announcement Management

| Card | USER & ACCESS | ANNOUNCEMENTS |
|---:|---|---|
| 30 | Permissions | Create Announcement |
| 31 | Permissions | Edit Announcement |
| 32 | Permissions | Archive Announcement |
| 33 | Permissions | Delete Announcement |

Full paths:

`USER & ACCESS → Authorization → Permissions → ANNOUNCEMENTS → Announcement Management → Create Announcement`

`USER & ACCESS → Authorization → Permissions → ANNOUNCEMENTS → Announcement Management → Edit Announcement`

`USER & ACCESS → Authorization → Permissions → ANNOUNCEMENTS → Announcement Management → Archive Announcement`

`USER & ACCESS → Authorization → Permissions → ANNOUNCEMENTS → Announcement Management → Delete Announcement`

---

## Package 12 — USER & ACCESS ↔ USER & ACCESS

### Authorization ↔ User Management

| Card | USER & ACCESS | USER & ACCESS |
|---:|---|---|
| 34 | Permissions | Create User |
| 35 | Permissions | Edit User |
| 36 | Permissions | Disable User |

Full paths:

`USER & ACCESS → Authorization → Permissions ↔ USER & ACCESS → User Management → Create User`

`USER & ACCESS → Authorization → Permissions ↔ USER & ACCESS → User Management → Edit User`

`USER & ACCESS → Authorization → Permissions ↔ USER & ACCESS → User Management → Disable User`

---

# 4. Relationship Key — Card Audit

| Package | Domain Pair | Category Pair | Cards |
|---:|---|---|---|
| 01 | BOOKS ↔ CONTRIBUTORS | People & Relations ↔ Roles | 01–04 |
| 02 | BOOKS ↔ CONTRIBUTORS | Book Management ↔ Contributions | 05 |
| 03 | ANNOUNCEMENTS ↔ MEDIA | Content ↔ Media Types | 06 |
| 04 | USER & ACCESS ↔ CONTRIBUTORS | Authorization ↔ Person Management | 07–10 |
| 05 | BOOKS ↔ MEDIA | Media ↔ Media Types | 11–13 |
| 06 | CONTRIBUTORS ↔ MEDIA | Person Information ↔ Media Types | 14 |
| 07 | USER & ACCESS ↔ MEDIA | Authorization ↔ Media Management | 15–19 |
| 08 | USER & ACCESS ↔ BOOKS | Authorization ↔ Book Management | 20–23 |
| 09 | ANNOUNCEMENTS ↔ BOOKS | Type ↔ Book Management / Publishing | 24–28 |
| 10 | ANNOUNCEMENTS ↔ MEDIA | Announcement Management ↔ Media Types | 29 |
| 11 | USER & ACCESS ↔ ANNOUNCEMENTS | Authorization ↔ Announcement Management | 30–33 |
| 12 | USER & ACCESS ↔ USER & ACCESS | Authorization ↔ User Management | 34–36 |

**Total relationship cards: 36**

---

# 5. High-Level Domain Connectivity

```text
                    ┌──────────────────────┐
                    │   USER & ACCESS      │
                    └──────────────────────┘
                       ↕      ↕      ↕
                      BOOKS  CONTRIBUTORS ANNOUNCEMENTS
                        ↕       ↕          ↕
                        └────── MEDIA ─────┘
```

The actual Relationship Key is more granular than this overview. Every mapped connection must be interpreted at:

`Domain → Category → Subcategory`

rather than only at Domain level.

---

# 6. Core Structural Rule

The System Map uses this hierarchy:

```text
DOMAIN
└── CATEGORY
    └── SUBCATEGORY
```

A relationship is therefore represented as:

```text
DOMAIN
→ CATEGORY
→ SUBCATEGORY
↔
DOMAIN
→ CATEGORY
→ SUBCATEGORY
```

When several Subcategories belong to the same Category pair, they remain inside one **Relationship Package**.

Example:

```text
BOOKS
→ People & Relations
    → Author
    → Translator
    → Editor
    → Illustrator

↔

CONTRIBUTORS
→ Roles
    → Author
    → Translator
    → Editor
    → Illustrator
```

This grouping rule prevents unrelated Subcategories from being artificially connected and keeps the Relationship Map readable and traceable.

