# BWL Data and Persistence

## Purpose

This document defines how BWL represents, stores, retrieves, updates, and manages business data.

The central principle is:

> Define once. Reuse everywhere. Let BWL handle persistence.

BWL should allow developers to describe business data once and use that same definition for forms, saving, tables, reports, workflows, queries, and database persistence.

The developer should not repeatedly define the same fields or manually map business fields to database fields.

## 1. Business Object as the Source of Truth

A business object is defined once.

entity Employee {
    id: auto
    name: required text
    email: required email
    department: text
    position: text
}

The `Employee` definition is the source of truth for the business data.

The same definition may be reused by:

form Employee

save Employee

table Employee

report Employee

workflow EmployeeApproval

get Employee

update Employee

delete Employee

No duplicate field declarations are required.

## 2. Complete Form Persistence

A form may contain many fields.

form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    address: text
    school: text
    course: text
    field8: text
    field9: text
    field10: text
}

The complete submitted form can be saved with:

save SchoolForm

This means:

> Save the complete submitted SchoolForm business data.

If the form contains 50 fields, BWL automatically handles the complete object.

The developer should not need to write:

save firstName
save lastName
save age
save location
save address
...
save field50

## 3. One Definition, One Mapping

BWL should not require a separate mapping for every field.

If the business object is:

entity Employee {
    id: auto
    name: text
    email: email
    department: text
}

then:

save Employee

is sufficient for normal persistence.

BWL understands the fields from the `Employee` definition.

The persistence layer determines how those fields are represented in the configured database.

## 4. Automatic Persistence

When BWL executes:

save Employee

the runtime handles the underlying persistence operation.

Depending on the state of the object, BWL may create a new record or update an existing record.

The developer does not normally need to manually write SQL, database connection code, insert statements, update statements, or field mappings.

## 5. Save as the High-Level Operation

`save` is the preferred high-level persistence operation.

save Employee

means:

> Persist the current state of the Employee business object.

BWL determines the appropriate persistence behavior.

For a new object, it may create a record.

For an existing object, it may update the existing record.

This avoids unnecessary create-versus-update logic in ordinary application code.

## 6. Create

BWL may explicitly request creation when the distinction is important.

create Employee

This means:

> Create a new Employee business record.

The complete structure still comes from the existing `Employee` definition.

The developer does not repeat the fields.

## 7. Read

Business data may be retrieved using:

get Employee

BWL determines the required persistence operation.

A filtered retrieval may be expressed as:

get Employee where Employee.department = "IT"

The business expression is translated by the compiler/runtime into the appropriate database operation.

## 8. Update

An existing business object may be updated using:

update Employee

The runtime determines the identity of the object and updates the corresponding persistent record.

The developer does not need to manually update each field.

## 9. Delete

A business object may be deleted using:

delete Employee

The operation is subject to BWL permissions, business rules, relationships, and configured deletion behavior.

## 10. Complete CRUD

BWL supports the normal business data lifecycle:

create Employee

get Employee

update Employee

delete Employee

For common cases, the unified operation:

save Employee

may handle creation or updating automatically.

## 11. Identity

Business objects may have an identity.

entity Employee {
    id: auto
    name: text
    email: email
}

The `id` identifies the persistent business record.

The runtime may generate the identifier automatically.

The developer should not normally need to manually create database-specific identifier logic.

## 12. Automatic Identifiers

A field may use automatic identity generation.

entity Employee {
    id: auto
    name: text
    email: email
}

When a new Employee is saved, BWL may automatically generate the identifier.

The generated identifier becomes part of the business object.

## 13. Required Fields

Fields may declare requirements.

entity Employee {
    id: auto
    name: required text
    email: required email
    department: text
}

BWL automatically knows that `name` and `email` are required.

The developer should not repeat those requirements in every form or save operation.

## 14. Type Information

Field types are defined once.

entity Employee {
    name: text
    age: number
    email: email
}

BWL understands:

name is text

age is numeric

email is an email value

The same type information is reused by forms, validation, persistence, tables, reports, and other interfaces.

## 15. Automatic Validation

Before persistence, BWL validates data according to the business definition.

For example:

entity Employee {
    name: required text
    age: number
    email: required email
}

If invalid data is submitted, the save operation should fail before invalid data is persisted.

Basic validation should not need to be duplicated inside every interface.

## 16. Business Rules

Additional business rules may be defined separately.

rule EmployeeValidation {
    if Employee.age < 18 {
        reject Employee
    }
}

The rule operates on the existing `Employee` business object.

It does not redefine the Employee fields.

## 17. Relationships

Business objects may reference other business objects.

entity Department {
    id: auto
    name: text
}

entity Employee {
    id: auto
    name: text
    department: Department
}

BWL understands that `Employee.department` references `Department`.

The persistence implementation handles the underlying relationship.

## 18. Relationship Reuse

Once a relationship exists:

Employee.department

it may be reused in:

forms

tables

reports

filters

rules

workflows

queries

interfaces

The relationship does not need to be declared again.

## 19. Nested Business Data

BWL may support nested business data.

entity Employee {
    id: auto
    name: text

    address {
        street: text
        city: text
        country: text
    }
}

The complete business object may be persisted using:

save Employee

The developer does not manually save each nested field.

## 20. Transactions

Multiple persistence operations may be grouped into a transaction.

transaction {
    save Employee
    save Department
}

The runtime manages commit and rollback behavior according to the persistence provider.

The developer should not normally manage database connections or transaction mechanics manually.

## 21. Transaction Safety

The persistence layer is responsible for maintaining transaction integrity.

BWL should handle:

connection management

commit

rollback

connection cleanup

transaction boundaries

The developer describes the business operation.

## 22. High-Level Queries

BWL may express business queries directly.

get Employee where Employee.department = "IT"

The developer describes the required business data.

The compiler/runtime translates the expression into the appropriate persistence operation.

## 23. Filtering

Filtering uses existing business fields.

get Employee where Employee.age >= 18

No database column declaration is required.

The field already exists in the `Employee` definition.

## 24. Sorting

Sorting uses existing business fields.

get Employee order by Employee.name

The field is already known from the business object.

## 25. Pagination

Large result sets may be paginated.

get Employee page 1 size 50

The persistence layer handles efficient retrieval of the requested records.

The application developer does not need to manually construct database-specific pagination queries for normal operations.

## 26. Database Schema

The BWL compiler may derive the required persistence schema from business definitions.

For example:

entity Employee {
    id: auto
    name: text
    email: email
    department: text
}

The compiler/runtime may generate or synchronize the required database representation.

The developer should not normally need to manually create every database table.

## 27. Schema Evolution

If the business object changes:

entity Employee {
    id: auto
    name: text
    email: email
    department: text
    position: text
}

BWL recognizes `position` as part of the updated business definition.

Existing operations such as:

form Employee

save Employee

table Employee

report Employee

automatically use the updated definition where the complete object is referenced.

The persistence system may generate or apply the required schema migration.

## 28. Business Definition as Source of Truth

The normal architecture is:

BWL Business Definition
        ↓
BWL Compiler
        ↓
Persistence Model
        ↓
Database

The database is an implementation target.

The business definition remains the source of truth.

## 29. Persistence Providers

BWL should support pluggable persistence providers.

Conceptually:

BWL
 │
 └── Persistence API
        ├── PostgreSQL Provider
        ├── MySQL Provider
        ├── SQLite Provider
        └── Other Providers

The business code should remain independent from the selected provider whenever the required capabilities are supported.

## 30. Database Independence

The following:

save Employee

has the same business meaning regardless of whether the configured provider uses PostgreSQL, MySQL, SQLite, or another supported database.

The implementation may change.

The business intent does not.

## 31. External Data Sources

BWL may support business objects backed by external services.

For example:

entity Customer from CustomerAPI

The rest of the BWL application may continue to work with the `Customer` business object.

The provider handles the communication with the external system.

## 32. Persistence Abstraction

BWL separates business intent from persistence implementation.

The developer describes:

> What business data should be saved or retrieved.

The persistence layer determines:

> How that data is stored or retrieved.

This keeps BWL business code independent from database-specific implementation details.

## 33. Permissions

Persistence operations must respect BWL security.

For example:

permission EditEmployee {
    role HR
}

Then:

update Employee requires permission EditEmployee

The permission system controls who can perform the operation.

The persistence layer must enforce the result of the authorization decision.

## 34. Audit Information

BWL may automatically maintain standard audit information such as:

createdAt

createdBy

updatedAt

updatedBy

These values should not require repetitive manual handling in every save operation.

## 35. Soft Delete

BWL may support configurable soft deletion.

For example:

entity Employee {
    id: auto
    name: text
    deleted: boolean
}

A high-level:

delete Employee

may perform a soft delete when the business object or persistence configuration requires it.

The application code remains unchanged.

## 36. Data Lifecycle

Business data may have a lifecycle.

For example:

Created
   ↓
Active
   ↓
Archived

The lifecycle may be defined through BWL rules or workflows.

Persistence stores the current state.

## 37. Form to Database Flow

A complete business flow may look like:

form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    address: text
    school: text
    course: text
}

save SchoolForm

The runtime handles:

Form Input
    ↓
Validation
    ↓
Business Object
    ↓
Persistence
    ↓
Database

The developer does not manually map every input field.

## 38. Complete Employee Example

entity Employee {
    id: auto
    name: required text
    email: required email
    age: number
    department: text
    position: text
}

The same definition can be reused:

form Employee

save Employee

table Employee

report Employee

get Employee

update Employee

delete Employee

One definition controls the complete business object.

## 39. No Duplicate Database Mapping

BWL should avoid requiring:

Form field → database field
Form field → database field
Form field → database field
...
Form field → database field

when the business object already defines those fields.

Instead:

save Employee

should be sufficient for normal persistence.

## 40. No Duplicate CRUD Implementation

BWL should avoid requiring repetitive functions such as:

insertEmployee()
updateEmployee()
deleteEmployee()
findEmployee()
findEmployees()

for standard business operations.

Instead:

create Employee

get Employee

update Employee

delete Employee

and:

save Employee

should provide the high-level operations.

## 41. Compiler Responsibilities

The BWL compiler should validate persistence declarations.

It should verify:

- Business objects exist
- Fields exist
- Field types are valid
- Relationships are valid
- Queries reference valid fields
- Persistence operations are valid
- Required permissions exist
- Persistence providers support required capabilities

The compiler should also translate high-level persistence declarations into the appropriate application/runtime representation.

## 42. Runtime Responsibilities

The BWL runtime should handle:

- Database connections
- Query execution
- Persistence
- Retrieval
- Updates
- Deletes
- Transactions
- Connection management
- Error handling
- Provider communication
- Runtime validation
- Persistence lifecycle

The developer should not manually implement these for normal BWL operations.

## 43. Error Handling

Persistence errors should be represented at the BWL level where possible.

For example:

try {
    save Employee
}
catch DatabaseError {
    notify "Employee could not be saved"
}

Technical details may remain available for debugging and logging.

## 44. Logging

Persistence operations may be logged automatically according to application configuration.

The developer should not need to manually add logging to every persistence operation.

## 45. Performance

The compiler and runtime should be able to optimize persistence operations.

For example:

table Employee.name, Employee.department

should not require unnecessary retrieval of unrelated fields when the persistence provider supports optimized field selection.

BWL describes the required business information.

The runtime determines an efficient implementation.

## 46. Data Integrity

The persistence layer should preserve:

- Field types
- Required fields
- Relationships
- Constraints
- Transaction boundaries
- Business rules where applicable

The database implementation should not silently violate the BWL business model.

## 47. Persistence and Interfaces

Forms, tables, reports, and interfaces should all use the same underlying business definitions.

For example:

entity Employee {
    id: auto
    name: text
    email: email
    department: text
}

Then:

form Employee

table Employee

report Employee

save Employee

All of these refer to the same business object.

There is no second Employee definition.

## 48. Persistence and Workflows

Workflows may operate on persisted business objects.

For example:

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

A task may save the current object:

save Employee

The workflow and persistence systems operate on the same business object.

No duplicate data structure is required.

## 49. Automatic Persistence Principle

The complete concept is:

Define Business Object
        ↓
BWL Understands Structure
        ↓
Form Uses It
        ↓
Validation Uses It
        ↓
Save Uses It
        ↓
Table Uses It
        ↓
Report Uses It
        ↓
Workflow Uses It
        ↓
Database Uses It

The same definition flows through the complete application.

## 50. Core Design Principle

BWL data and persistence follow the same fundamental rule:

> Define once. Reuse everywhere.

The developer describes the business object.

BWL handles the repetitive implementation details.

This includes:

- Field mapping
- Type handling
- Validation
- Persistence
- CRUD
- Queries
- Transactions
- Database interaction
- Schema generation
- Schema evolution
- Provider abstraction

## 51. BWL Goal

The goal is not to create a shorter syntax for SQL.

The goal is to allow the developer to express the business requirement directly.

Instead of:

Define fields
↓
Create database mappings
↓
Create insert logic
↓
Create update logic
↓
Create validation
↓
Create queries
↓
Create connection handling
↓
Create transaction handling

BWL should allow:

entity Employee {
    id: auto
    name: required text
    email: required email
    department: text
    position: text
}

save Employee

The compiler and runtime handle the underlying implementation.

## 52. Summary

BWL treats business objects as the source of truth.

For example:

entity Employee {
    id: auto
    name: required text
    email: required email
    department: text
    position: text
}

From this single definition, BWL can support:

form Employee

save Employee

table Employee

report Employee

get Employee

update Employee

delete Employee

The developer does not repeatedly describe the same fields.

The compiler and runtime translate the high-level business operations into the required persistence implementation.

The central rule remains:

> Define once. Reuse everywhere. Let BWL handle the details.

## 53. Final Principle

BWL persistence must always preserve the main language philosophy:

> **One business definition. One source of truth. Many automatic uses.**

If a normal operation requires the developer to repeat information that BWL already knows, that operation should be reconsidered and simplified.

The ultimate goal is:

> **Less code. Less repetition. More business meaning.**
