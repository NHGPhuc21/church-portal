# IEUDC Church Portal - Development Specification

## Overview

**IEUDC (International Evangelical United District Churches)** is a digital portal to manage church data and engagement for all levels—local, district, national, and international. It serves as a centralized platform for managing members, events, and hierarchical church structures.

## Core Features

* Church hierarchy and relationships
* Member management
* Role-based access (Admin / Contributor)
* Event creation and attendance tracking
* Change logging (audit trail)

## Data Models

### 1. Church

Represents a church entity in the hierarchy.

| Field         | Type        | Description                                                         |
| ------------- | ----------- | ------------------------------------------------------------------- |
| `id`          | UUID / ID   | Primary key                                                         |
| `code`        | String      | Unique identifier code (e.g. LOC-MNL-001)                           |
| `name`        | String      | Full church name                                                    |
| `description` | Text        | Description of the church                                           |
| `type`        | Enum        | One of: `Mission`, `Local`, `District`, `National`, `International` |
| `parent_id`   | FK → Church | References the parent church for hierarchy                          |
| `created_at`  | Timestamp   |                                                                     |
| `updated_at`  | Timestamp   |                                                                     |

**Relationships:**

* Has many members
* Has many pastors (members tagged as pastor)
* Belongs to parent church
* Has many child churches

### 2. User

Platform user with access to one or more churches.

| Field        | Type          | Description                           |
| ------------ | ------------- | ------------------------------------- |
| `id`         | UUID / ID     | Primary key                           |
| `name`       | String        | Full name                             |
| `email`      | String        | Unique email                          |
| `password`   | Hashed String | Securely stored password              |
| `role`       | Enum          | `Admin`, `Contributor`                |
| `church_ids` | Many-to-many  | Churches this user is associated with |
| `created_at` | Timestamp     |                                       |
| `updated_at` | Timestamp     |                                       |

**Relationships:**

* Belongs to many churches
* Can create/update events, members (if authorized)

### 3. Member

Church members belonging to local or mission churches.

| Field          | Type          | Description                             |
| -------------- | ------------- | --------------------------------------- |
| `id`           | UUID / ID     | Primary key                             |
| `church_id`    | FK → Church   | Local or Mission church assignment      |
| `first_name`   | String        |                                         |
| `last_name`    | String        |                                         |
| `birthdate`    | Date          |                                         |
| `address`      | Text          |                                         |
| `baptism_date` | Date          |                                         |
| `ministries`   | Array / JSON  | Ministries involved in                  |
| `assignation`  | String / Enum | Current assignment (e.g., usher, choir) |
| `is_pastor`    | Boolean       | Tagged as pastor                        |
| `created_at`   | Timestamp     |                                         |
| `updated_at`   | Timestamp     |                                         |

**Relationships:**

* Belongs to a church
* May attend many events
* May facilitate events

### 4. Event

Tracks events across different church levels.

| Field         | Type        | Description                                      |
| ------------- | ----------- | ------------------------------------------------ |
| `id`          | UUID / ID   | Primary key                                      |
| `title`       | String      | Event name                                       |
| `description` | Text        | Full description                                 |
| `church_id`   | FK → Church | Church responsible for the event                 |
| `level`       | Enum        | `Local`, `District`, `National`, `International` |
| `start_date`  | DateTime    | Start of event                                   |
| `end_date`    | DateTime    | End of event                                     |
| `created_by`  | FK → User   | Creator of the event                             |
| `created_at`  | Timestamp   |                                                  |
| `updated_at`  | Timestamp   |                                                  |

#### Submodels / Join Tables:

**EventMembers**

* `event_id` → Event
* `member_id` → Member
* `is_facilitator` → Boolean

**Relationships:**

* Belongs to a church
* Has many members (attendees)
* Has facilitators (subset of members)

### 5. Log

Audit trail for tracking system-wide changes.

| Field        | Type      | Description                             |
| ------------ | --------- | --------------------------------------- |
| `id`         | UUID / ID | Primary key                             |
| `user_id`    | FK → User | Who performed the action                |
| `action`     | String    | e.g. `create_member`, `update_event`    |
| `model_type` | String    | Model affected (e.g. `Member`, `Event`) |
| `model_id`   | UUID / ID | ID of the record affected               |
| `changes`    | JSON      | What changed (diff)                     |
| `timestamp`  | Timestamp | When it happened                        |

## Permissions

| Action                  | Admin | Contributor      |
| ----------------------- | ----- | ---------------- |
| Manage churches         | ✅     | ❌                |
| Manage members          | ✅     | ✅ (within scope) |
| Manage events           | ✅     | ✅ (within scope) |
| Assign member as pastor | ✅     | ✅                |
| View reports / logs     | ✅     | ❌                |
| Manage users / roles    | ✅     | ❌                |

## Additional Notes

* **Church Tree:** Use recursive relationships in `Church` (`parent_id`) for hierarchy.
* **Scoping:** Contributors can only manage data tied to their associated churches.
* **Member Lookup:** Optional global search with church filters.
* **Notifications:** Optional event/member notifications.
* **API Ready:** Consider RESTful or GraphQL API for future extensions.

## Development Stack (Recommended)

| Layer      | Tech Suggestions                     |
| ---------- | ------------------------------------ |
| Backend    | Laravel / NestJS                     |
| Frontend   | React / InertiaJS / Vue              |
| Auth       | Sanctum / Passport / JWT             |
| Database   | PostgreSQL / MySQL                   |
| Logs       | DB-stored or via Spatie activity log |
| Deployment | Docker + CI/CD + Laravel Vapor/Forge |
