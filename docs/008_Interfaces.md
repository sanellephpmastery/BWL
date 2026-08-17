# BWL Interfaces

## Purpose

This document defines interfaces in BWL (Business Workflow Language).

BWL interfaces provide a simple way to define business data and expose that data to forms, tables, reports, workflows, users, and external systems.

The central principle of BWL is:

> **Define once. Reuse everywhere. Let BWL handle the details.**

BWL should minimize repetitive declarations and implementation code.

## 1. Single Source of Truth

A business object should be defined only once.

For example:

```bwl
form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    address: text
    school: text
    course: text
    ...
}
```

The `SchoolForm` definition contains the complete structure of the business data.

Other BWL statements should reference `SchoolForm` instead of redefining its fields.

## 2. Form as a Business Object

A form may define the complete structure of information that a business process needs.

```bwl
form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    address: text
    school: text
    course: text
}
```

The form definition can contain many fields.

The number of fields does not change how the form is referenced.

Once the form is defined, BWL knows its complete structure.

## 3. Saving a Complete Form

A complete form can be saved by referencing its name.

```bwl
save SchoolForm
```

This means:

> Save the complete submitted `SchoolForm` data.

The developer does not need to list every field again.

For example, if `SchoolForm` contains 50 fields:

```bwl
save SchoolForm
```

automatically applies to all 50 fields.

There should be no need to write:

```text
save firstName
save lastName
save age
save location
...
save field50
```

## 4. Automatic Persistence

When BWL saves a business object, the runtime determines how the data is persisted.

For example:

```bwl
save SchoolForm
```

may result in the equivalent of database operations internally.

The BWL developer does not normally need to write SQL or database-specific persistence code.

The runtime is responsible for:

* Mapping fields
* Validating values
* Creating records
* Updating records
* Handling persistence
* Maintaining relationships
* Reporting persistence errors

The business definition remains independent from the database technology.

## 5. Reusing a Form

Once defined, the same business object can be referenced anywhere.

```bwl
form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    school: text
}
```

It may then be used as:

```bwl
save SchoolForm
```

```bwl
table SchoolForm
```

```bwl
report SchoolForm
```

```bwl
view SchoolForm
```

```bwl
workflow SchoolApproval
```

The fields do not need to be declared again.

## 6. Automatic Tables

A table may reference the complete business object.

```bwl
table SchoolForm
```

This means:

> Display the available data using the fields defined by `SchoolForm`.

If the form contains 50 fields, the table knows that the object contains those 50 fields.

No second field declaration is required.

## 7. Selecting Specific Fields

A table may optionally select specific fields.

```bwl
table SchoolForm.firstName
```

This displays only the `firstName` field.

Multiple fields may be selected:

```bwl
table SchoolForm.firstName, SchoolForm.lastName, SchoolForm.school
```

The field definitions still come from `SchoolForm`.

## 8. Field References

The notation:

```bwl
SchoolForm.firstName
```

means:

> The `firstName` field belonging to `SchoolForm`.

Examples:

```bwl
SchoolForm.lastName
SchoolForm.age
SchoolForm.location
SchoolForm.school
```

Field references may be used in:

* Tables
* Reports
* Conditions
* Rules
* Workflows
* Interfaces
* Filters
* Sorting
* Business calculations

## 9. Automatic Forms

A business object may be used to generate a form automatically.

For example, if an entity is defined:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
    position: text
}
```

then:

```bwl
form Employee
```

means:

> Generate a form using the Employee definition.

The developer does not need to repeat:

```text
input id
input name
input email
input department
input position
```

The entity already contains that information.

## 10. Entity and Form Relationship

Entities define reusable business data.

Forms provide a way to collect or edit that data.

For example:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
}
```

A form may use it:

```bwl
form Employee
```

The entity remains the source of truth for the field definitions.

## 11. Automatic Validation

BWL uses the existing type definitions when handling forms.

For example:

```bwl
entity Employee {
    name: text
    age: number
    email: email
}
```

The system automatically knows:

* `name` must be text
* `age` must be a number
* `email` must be an email value

The developer does not need to repeat those validations inside the form.

Additional business rules may still be defined separately.

```bwl
rule EmployeeValidation {
    if Employee.age < 18 {
        reject Employee
    }
}
```

## 12. Automatic CRUD Operations

BWL may provide standard business operations.

```bwl
create Employee
```

```bwl
view Employee
```

```bwl
update Employee
```

```bwl
delete Employee
```

These operations automatically use the known structure of `Employee`.

The developer does not need to specify every field for each operation.

## 13. Automatic Updates

If a business object changes, references to that object should automatically recognize the new structure.

For example:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
}
```

Later:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
    position: text
}
```

The existing:

```bwl
form Employee
```

and:

```bwl
table Employee
```

should automatically recognize `position`.

The developer should not need to modify each interface separately.

## 14. Automatic Workflow Routing

BWL should describe normal workflow routing directly.

For example:

```bwl
workflow SchoolApproval {
    Tasker -> Reviewer -> MasterReviewer
}
```

This means:

```text
Tasker
   ↓
Reviewer
   ↓
MasterReviewer
```

When `Tasker` completes, the workflow automatically moves to `Reviewer`.

When `Reviewer` completes, it automatically moves to `MasterReviewer`.

The developer does not need to manually write transition logic.

## 15. No Repetitive Routing Code

The following should normally not be required for a simple sequential workflow:

```bwl
if Tasker.done {
    send Reviewer
} else if Reviewer.done {
    send MasterReviewer
}
```

Instead:

```bwl
Tasker -> Reviewer -> MasterReviewer
```

is enough.

The BWL runtime handles the transition mechanics.

## 16. Workflow Participants

Participants may be defined as roles.

```bwl
role Tasker
role Reviewer
role MasterReviewer
```

Then:

```bwl
workflow SchoolApproval {
    Tasker -> Reviewer -> MasterReviewer
}
```

The workflow defines the sequence.

The permission and assignment systems determine who may perform each role.

## 17. Conditional Routing

Direct routing should be preferred when the process is sequential.

When a real business decision exists, BWL may use a condition.

For example:

```bwl
if SchoolForm.age >= 18 {
    approve SchoolForm
} else {
    reject SchoolForm
}
```

The purpose of `if` is to express a genuine business decision.

It should not be required merely to move from one workflow task to the next.

## 18. Interface Actions

Interfaces may expose business operations by referencing existing objects.

```bwl
interface SchoolManagement {
    create SchoolForm
    view SchoolForm
    update SchoolForm
}
```

The interface does not redefine the fields of `SchoolForm`.

## 19. Permissions

Permissions control who may perform protected operations.

```bwl
permission ApproveSchool {
    role Reviewer
}
```

The interface may reference the permission:

```bwl
approve SchoolForm
    requires permission ApproveSchool
```

Authorization is handled by the permission system.

The interface should not duplicate authorization logic.

## 20. Filters

Interfaces may filter existing business data.

```bwl
table Employee
    where Employee.department = "IT"
```

The filter references the existing field.

No new field definition is necessary.

## 21. Sorting

Interfaces may sort existing business data.

```bwl
table Employee
    order by Employee.name
```

The `name` field already belongs to `Employee`.

## 22. Reports

Reports may reuse complete business objects.

```bwl
report SchoolReport {
    SchoolForm
}
```

This means the report can use the structure already defined by `SchoolForm`.

Specific fields may also be selected:

```bwl
report SchoolReport {
    SchoolForm.firstName
    SchoolForm.lastName
    SchoolForm.school
}
```

## 23. External Systems

External systems may interact with BWL business objects through interfaces.

For example:

```bwl
interface SchoolAPI {
    create SchoolForm
    view SchoolForm
    update SchoolForm
}
```

The external implementation may use a web API, application, mobile system, or another technology.

The BWL business definition remains the same.

## 24. Database Independence

BWL business definitions should not depend on a specific database.

For example:

```bwl
save SchoolForm
```

does not specify:

* SQL
* MySQL
* PostgreSQL
* MongoDB
* SQLite
* File storage

The persistence implementation is handled separately.

The BWL program expresses the business requirement:

> Save this complete business object.

## 25. Automatic Relationship Handling

If business objects reference other objects, BWL should preserve those relationships.

Example:

```bwl
entity Student {
    name: text
    school: School
}
```

The system understands that `Student.school` references a `School`.

The developer should not need to manually implement the underlying relationship every time it is used.

## 26. Define Once, Reuse Everywhere

The same definition may be used across the system.

Example:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
    position: text
}
```

Then:

```bwl
form Employee
```

```bwl
save Employee
```

```bwl
table Employee
```

```bwl
report Employee
```

```bwl
workflow EmployeeApproval
```

The business data remains defined only once.

## 27. Complete School Example

A complete example may look like:

```bwl
form SchoolForm {
    firstName: text
    lastName: text
    age: number
    location: text
    address: text
    school: text
    course: text
}
```

The same object may then be used:

```bwl
form SchoolForm
```

```bwl
save SchoolForm
```

```bwl
table SchoolForm
```

```bwl
report SchoolForm
```

And the approval process:

```bwl
workflow SchoolApproval {
    Tasker -> Reviewer -> MasterReviewer
}
```

No duplicate 50-field declarations are required.

No repetitive database code is required.

No repetitive routing code is required.

## 28. Core BWL Philosophy

BWL should follow these principles:

### Define Once

A business object is defined in one place.

### Reuse Everywhere

The object can be referenced by name throughout the system.

### Automatic Understanding

BWL knows the fields, types, relationships, and business meaning of the referenced object.

### Automatic Persistence

Saving an object saves its complete data.

### Automatic Presentation

Tables, forms, and reports can derive their structure from the object.

### Automatic Routing

Sequential workflows can move between roles without repetitive control-flow code.

### Explicit Business Decisions

`if` and other conditions remain available for genuine business decisions.

### Minimal Repetition

The developer should not repeatedly describe information that BWL already knows.

## 29. Design Principle

The goal of BWL is not to make programmers write shorter versions of traditional code.

The goal is to allow business intent to be expressed directly.

Instead of:

```text
Define 50 fields
↓
Define 50 database mappings
↓
Define 50 save operations
↓
Define 50 display fields
↓
Write routing code
↓
Write validation code
```

BWL should allow:

```bwl
form SchoolForm {
    ... business fields ...
}

save SchoolForm

table SchoolForm

workflow SchoolApproval {
    Tasker -> Reviewer -> MasterReviewer
}
```

BWL understands the relationships between these declarations and handles the repetitive implementation details.

## 30. Summary

BWL interfaces are based on a simple idea:

> **The developer describes the business object once.**

After that, BWL should be able to reuse that definition for:

* Forms
* Saving
* Database persistence
* Tables
* Reports
* Validation
* CRUD operations
* Workflows
* Permissions
* External interfaces

The central BWL philosophy is:

> **Define once. Reference by name. Reuse everywhere. Let BWL handle the rest.**
