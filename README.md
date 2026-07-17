# Building Management — Django Data Model

A database-first Django project for modelling the operational, financial, and communication workflows of a residential building.

The project explores how residents, building managers, and business owners interact around a shared building: managers approve services, residents submit requests and maintenance issues, payments and utility bills are recorded, and the community communicates through announcements, messages, notifications, surveys, and reminders.

> This repository is primarily a **domain-modelling and database-design project**. Its main focus is the relational schema and its implementation with the Django ORM.

![Building Management database diagram](building/database.png)

## What this project demonstrates

- Translating a real-world business domain into **21 Django models**
- Designing one-to-one, one-to-many, and many-to-many relationships
- Extending Django's authentication model with a role-aware `CustomUser`
- Separating user identity from role-specific profiles
- Enforcing data integrity through foreign keys, unique fields, choices, and cascading rules
- Modelling stateful workflows for services, issues, contracts, and notifications
- Using Django permissions for role-sensitive operations
- Evolving a schema through Django migrations

## Domain overview

The system has three principal actors:

| Actor | Responsibility |
| --- | --- |
| Resident | Belongs to a building as an owner, tenant, or visitor; requests services and reports issues |
| Building Manager | Manages one building, approves services, handles operations, and can be verified by an administrator |
| Business Owner | Provides services that can be reviewed and activated for residents |

The database is organized around six connected areas:

1. **Identity and access** — custom users, roles, groups, permissions, and role-specific profiles
2. **Building governance** — buildings, managers, manager verification, management transfers, and contracts
3. **Service marketplace** — business-provided services, manager activation, availability windows, and resident requests
4. **Financial management** — subscriptions, payments, utility bills, unique receipts, and per-user wallets
5. **Community communication** — announcements, direct messages, reminders, and delivery-tracked notifications
6. **Resident engagement** — surveys, survey responses, FAQs, and maintenance issue reports

## Data-model highlights

### Role-aware identity

`CustomUser` extends Django's `AbstractUser` and introduces resident, manager, and business roles. One-to-one profile tables keep role-specific data separate:

- `Resident` connects a user to a building and classifies them as owner, tenant, or visitor.
- `BuildingManager` connects a manager to exactly one building and tracks verification and admin approval.
- `BusinessOwner` stores business-specific identity and verification state.

This design avoids placing unrelated nullable fields on a single user table while preserving Django's authentication and permission system.

### Explicit relationship cardinality

The schema uses relationship types to express business rules:

- **One-to-one:** user profiles, one manager per building, one wallet per user, and one bill record per payment
- **One-to-many:** a building's residents, announcements, surveys, subscriptions, bills, and transfer requests
- **Many-to-many:** Django groups and permissions assigned through the custom user model
- **Self-domain relationships:** direct messages reference `CustomUser` twice, with distinct `sent_messages` and `received_messages` reverse relations

### Workflow and state modelling

Business processes are represented with controlled state fields rather than free-form values:

- Service requests: `pending → approved → completed/cancelled`
- Maintenance issues: `open → in_progress → resolved`
- Contracts: `pending → approved`
- Notifications: `sent / failed`
- Services and surveys: active/inactive lifecycle flags

Django `choices` keep these states consistent at both the ORM and application layers.

### Financial integrity

- Money is stored with `DecimalField`, avoiding floating-point storage errors.
- Payment receipt numbers are unique.
- Bill payments link to both the building and the user who paid.
- Wallets are unique per user and encapsulate deposit and withdrawal behaviour.
- Subscriptions retain explicit date ranges and amounts.

### Referential integrity

Foreign keys and one-to-one relations use deliberate cascading deletion, keeping dependent records consistent when their owning domain object is removed. Named reverse relations on ambiguous relationships make ORM queries readable and predictable.

## Model catalogue

| Area | Models |
| --- | --- |
| Authentication | `CustomUser`, `Resident`, `BuildingManager`, `BusinessOwner` |
| Governance | `Building`, `ManagementTransferRequest`, `Contract` |
| Services | `Service`, `ServiceRequest` |
| Finance | `Subscription`, `Payment`, `BillPayment`, `Wallet` |
| Communication | `Announcement`, `Reminder`, `Message`, `Notification` |
| Engagement and support | `Survey`, `SurveyResponse`, `IssueReport`, `FAQ` |

## Django implementation

The project uses:

- Django ORM models and migrations
- A custom user model based on `AbstractUser`
- Django authentication and permission decorators
- Custom model permissions such as `can_activate_service` and `can_create_announcement`
- Function-based views for authentication, manager verification, services, payments, issue reporting, and wallet operations
- SQLite for the development database
- `django-extensions` for development tooling

The main schema is defined in [`building/app/models.py`](building/app/models.py), and its evolution is recorded in [`building/app/migrations/`](building/app/migrations/).

## Repository structure

```text
Building_management/
├── building/
│   ├── app/
│   │   ├── migrations/       # Versioned database schema changes
│   │   ├── models.py         # Domain and relational model definitions
│   │   └── views.py          # Django application workflows
│   ├── building/
│   │   ├── settings.py       # Project configuration
│   │   └── urls.py           # URL routing
│   ├── database.png          # Database diagram
│   ├── db.sqlite3            # Development database
│   └── manage.py
└── README.md
```

## Design decisions

- **Profile tables over a monolithic user table:** each actor stores only the fields relevant to its role.
- **A central building aggregate:** operational and community records are anchored to the building they belong to.
- **Normalized transactions:** payments and bill-specific information are separated, allowing the payment concept to be reused.
- **Auditable records:** timestamps, receipt identifiers, verification flags, and statuses preserve important operational history.
- **Database-backed authorization:** Django roles, groups, permissions, and custom permissions support separation of responsibilities.

## Project scope

This repository presents the data model and an early Django application layer. It is intended to demonstrate schema design, ORM relationship modelling, migrations, authentication concepts, and domain analysis rather than a production-ready deployment.

Potential next steps include REST API endpoints, database constraints for cross-table role validation, automated tests, transactional wallet operations, PostgreSQL support, environment-based settings, and a complete user interface.

## Author

**Amirali Taherdoust**

[GitHub profile](https://github.com/amirali-taherdoust)
