# BWL Validation and Rules

## Purpose

This document defines validation and business rules in BWL.

The central principle is:

> Define a business rule once. Reuse it everywhere.

Validation should not be repeatedly written inside forms, save operations, tables, reports, workflows, or database code.

BWL should understand the business rules from one central definition and automatically apply them where required.

## 1. Business Rules as the Source of Truth

A business rule is defined once.

For example:

rule AdultEmployee {
    Employee.age >= 18
}

The same rule may be reused by:

- Forms
- Save operations
- Workflows
- APIs
- Interfaces
- Reports
- Other business rules

The developer should not duplicate the same condition in every part of the application.

## 2. Field Validation

A field may define its basic validation directly.

entity Employee {
    id: auto
    name: required text
    age: number
    email: required email
}

BWL automatically understands:

- `name` is required
- `age` must be numeric
- `email` is required
- `email` must contain a valid email value

The same information can be reused by forms and persistence.

## 3. Required Fields

A required field may be declared as:

name: required text

When the value is missing, the business object is invalid.

For example:

entity Employee {
    name: required text
    email: required email
}

The following should not be accepted:

Employee {
    name: ""
    email: ""
}

The validation system should report the missing required values.

## 4. Type Validation

BWL validates values according to their declared type.

For example:

entity Employee {
    age: number
    name: text
    email: email
}

The following is invalid:

age = "hello"

because `age` is defined as a number.

The developer should not need to repeat this type check in every form.

## 5. Email Validation

An email field automatically carries email validation.

entity Employee {
    email: email
}

An invalid value should be rejected before persistence.

The developer does not need to write a separate email-validation function for every form.

## 6. Length Validation

Fields may define length constraints.

entity Employee {
    name: text min 2 max 100
}

The runtime validates the value against the declared limits.

The same validation can be reused by all interfaces that use `Employee.name`.

## 7. Numeric Validation

Numeric fields may define limits.

entity Employee {
    age: number min 18 max 100
}

BWL automatically understands that the value must fall within the configured range.

## 8. Business Rules

Business rules are different from basic field validation.

A field rule may say:

age must be a number.

A business rule may say:

Employee must be at least 18 years old.

Example:

rule AdultEmployee {
    Employee.age >= 18
}

Business rules operate at the business level.

## 9. Rejecting Invalid Data

A rule may reject a business object.

rule AdultEmployee {
    if Employee.age < 18 {
        reject Employee
    }
}

If the rule fails, the operation should not continue.

For example:

save Employee

should fail when the Employee violates the rule.

## 10. Validation During Save

When:

save Employee

is executed, BWL should automatically perform applicable validation before persistence.

The normal flow is:

Business Object
    ↓
Field Validation
    ↓
Business Rules
    ↓
Permission Check
    ↓
Persistence
    ↓
Database

The developer should not manually call every validation function.

## 11. Form Validation

A form using an existing business object automatically inherits its validation.

Example:

entity Employee {
    name: required text
    email: required email
    age: number min 18
}

Then:

form Employee

automatically understands the validation requirements.

The developer should not redefine:

required name

required email

age minimum

inside the form.

## 12. One Validation Definition

Avoid:

form Employee {
    name: required text
}

and separately:

save Employee {
    validate name
}

and separately:

api Employee {
    validate name
}

when the business object already defines the requirement.

Instead:

entity Employee {
    name: required text
}

All other components reuse that definition.

## 13. Cross-Field Validation

Some rules depend on multiple fields.

For example:

entity Employee {
    age: number
    position: text
}

A business rule may be:

rule SeniorPositionAge {
    if Employee.position = "Manager" and Employee.age < 21 {
        reject Employee
    }
}

The rule evaluates multiple business fields together.

## 14. Conditional Rules

Rules may use conditions.

rule EmployeeDepartment {
    if Employee.department = "IT" {
        require Employee.position
    }
}

The rule can require additional information based on existing business data.

## 15. Rule Messages

Rules may provide meaningful validation messages.

rule AdultEmployee {
    if Employee.age < 18 {
        reject Employee with "Employee must be at least 18 years old"
    }
}

The message may be displayed by the interface or returned through an API.

The rule remains defined only once.

## 16. Validation Errors

Validation errors should identify:

- Business object
- Field
- Rule
- Error message

Example:

Employee.age

Error:

Employee must be at least 18 years old.

The same error model can be used by forms, APIs, workflows, and other interfaces.

## 17. Multiple Validation Errors

BWL may return multiple validation errors at once.

For example:

entity Employee {
    name: required text
    email: required email
    age: number min 18
}

If all three values are invalid, the validation system may return all applicable errors rather than stopping after the first one.

## 18. Rule Reuse

A rule may be reused by multiple operations.

rule AdultEmployee {
    Employee.age >= 18
}

The rule may be used by:

form Employee

save Employee

workflow EmployeeApproval

api Employee

The developer does not duplicate the condition.

## 19. Explicit Rule Application

A rule may optionally be explicitly attached to a business object.

validate Employee with AdultEmployee

This may be useful when the application needs a specific validation set.

However, rules that are defined as mandatory business constraints should be applied automatically.

## 20. Mandatory Rules

A mandatory rule is always enforced when the business object is used in the relevant operation.

For example:

rule EmployeeEmailRequired {
    required Employee.email
}

Then saving Employee should automatically enforce the rule.

## 21. Optional Rules

Some rules may only apply to specific operations.

For example:

rule EmployeeCreationRules {
    on create Employee {
        require Employee.name
        require Employee.email
    }
}

Another rule may apply only during updates.

rule EmployeeUpdateRules {
    on update Employee {
        require Employee.id
    }
}

This allows BWL to distinguish different business lifecycle operations.

## 22. Create Rules

Rules may apply when a new object is created.

rule NewEmployeeRules {
    on create Employee {
        require Employee.name
        require Employee.email
    }
}

The rule does not need to be repeated inside the create operation.

## 23. Update Rules

Rules may apply when an existing object is updated.

rule EmployeeUpdateRules {
    on update Employee {
        require Employee.id
    }
}

The runtime determines whether the operation is a create or update.

## 24. Delete Rules

Rules may also control deletion.

rule EmployeeDeleteRules {
    on delete Employee {
        if Employee.status = "active" {
            reject Employee with "Active employees cannot be deleted"
        }
    }
}

This allows business policies to be enforced centrally.

## 25. Workflow Rules

Validation may also be applied during workflow transitions.

Example:

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

A rule may require:

rule ReviewerSubmission {
    when EmployeeApproval moves to Reviewer {
        require Employee.email
        require Employee.department
    }
}

The workflow uses the existing business object and validation rules.

## 26. Validation and Persistence

Validation occurs before persistence.

The normal flow is:

save Employee

becomes:

save Employee
    ↓
validate Employee
    ↓
apply business rules
    ↓
check permissions
    ↓
persist Employee

If validation fails:

persist Employee

does not occur.

## 27. Validation and Database Integrity

BWL validation should happen before data reaches the persistence layer.

However, database constraints may still exist as a second layer of protection.

The business layer should not depend solely on database errors to detect normal validation problems.

## 28. Business Rule Priority

Rules may have different priorities when necessary.

Example:

rule EmployeeBasicValidation priority 1 {
    ...
}

rule EmployeeApprovalValidation priority 2 {
    ...
}

The exact execution mechanism may be determined by the BWL runtime.

The important principle is that rule execution remains predictable.

## 29. Rule Dependencies

A rule may depend on another rule.

Example:

rule EmployeeBasicValidation {
    ...
}

rule EmployeeApprovalValidation {
    requires EmployeeBasicValidation
    ...
}

This allows larger business policies to be composed from reusable rules.

## 30. Rule Composition

Multiple rules may be combined.

validate Employee with:

    EmployeeBasicValidation
    AdultEmployee
    EmployeeDepartmentRule

The business rules remain independently defined and reusable.

## 31. Validation Groups

Applications may define validation groups for different contexts.

For example:

validation CreateEmployee {
    EmployeeBasicValidation
    NewEmployeeRules
}

validation ApproveEmployee {
    EmployeeBasicValidation
    EmployeeApprovalRules
}

The same business object may therefore have different validation requirements depending on the business operation.

## 32. Interface Validation

Interfaces should consume the validation model rather than recreate it.

For example:

form Employee

should automatically know:

- Required fields
- Field types
- Length constraints
- Numeric limits
- Email requirements
- Business validation rules

The interface can then display the appropriate validation messages.

## 33. API Validation

APIs should use the same validation model.

If an API receives:

Employee

the same business validation should apply.

The developer should not create a second validation implementation specifically for the API.

## 34. Table Editing Validation

If a table allows editing:

table Employee editable

the table should use the same validation rules as:

form Employee

save Employee

No duplicate validation definitions are required.

## 35. Report Validation

Reports generally read business data rather than create it.

However, report filters should still respect the business object's valid fields and types.

For example:

report Employee
    where Employee.age >= 18

The compiler should verify that:

- `Employee` exists
- `age` exists
- `age` is compatible with numeric comparison

## 36. Query Validation

Queries must use valid business fields.

This is valid:

get Employee where Employee.age >= 18

This should be rejected:

get Employee where Employee.unknownField = 10

The compiler should detect that `unknownField` does not exist.

## 37. Compile-Time Validation

Whenever possible, BWL should detect invalid rules during compilation.

For example:

entity Employee {
    age: number
}

rule InvalidRule {
    Employee.age = "hello"
}

The compiler should detect the incompatible comparison.

This prevents avoidable runtime errors.

## 38. Runtime Validation

Some validation cannot be completed until runtime.

For example:

entity Employee {
    email: email
}

Whether an email is already used by another Employee requires access to persisted data.

A runtime rule may therefore check:

rule UniqueEmployeeEmail {
    require unique Employee.email
}

The runtime performs the necessary lookup.

## 39. Unique Values

Business fields may require uniqueness.

entity Employee {
    email: unique email
}

BWL understands that two Employee records should not normally have the same email value.

The persistence layer may also enforce the uniqueness constraint.

## 40. Conditional Uniqueness

Uniqueness may depend on business conditions.

For example:

rule UniqueActiveEmployeeEmail {
    require unique Employee.email where Employee.status = "active"
}

The exact implementation may be handled by the persistence provider and runtime.

## 41. Validation and Security

Validation does not replace authorization.

For example:

Employee.age >= 18

answers:

> Is this data valid?

A permission answers:

> Is this user allowed to perform this operation?

These concerns remain separate.

## 42. Validation and Permissions

The normal order may be:

Business Object
    ↓
Validation
    ↓
Authorization
    ↓
Persistence

The exact runtime ordering may be optimized, but both validation and authorization must be enforced.

## 43. Validation and Transactions

If multiple business objects are processed together:

transaction {
    save Employee
    save Department
}

each operation should be validated before the transaction is successfully committed.

If a required rule fails, the transaction should not leave partial invalid state.

## 44. Validation Across Relationships

Rules may validate related business objects.

For example:

entity Department {
    id: auto
    name: required text
}

entity Employee {
    id: auto
    name: required text
    department: Department
}

A rule may require:

rule EmployeeDepartmentRequired {
    require Employee.department
}

The rule operates on the existing relationship.

## 45. Business Invariants

A business invariant is a condition that should always remain true.

Example:

rule EmployeeMustHaveDepartment {
    require Employee.department
}

The invariant should be enforced wherever the Employee is persisted or otherwise changed.

This prevents different interfaces from creating inconsistent data.

## 46. State Validation

Business objects may have states.

entity Employee {
    id: auto
    name: text
    status: text
}

A rule may control allowed states.

rule EmployeeStatus {
    Employee.status in ["active", "inactive", "archived"]
}

Invalid status values are rejected.

## 47. State Transition Rules

Rules may control transitions.

rule EmployeeArchive {
    when Employee.status changes from "active" to "archived" {
        require Employee.id
    }
}

This prevents arbitrary state changes when the business process requires restrictions.

## 48. Rule and Workflow Separation

A workflow controls:

> What process happens next.

A rule controls:

> Whether the business state is valid.

For example:

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

and:

rule EmployeeApprovalData {
    require Employee.email
}

The workflow and validation system cooperate without duplicating the business data definition.

## 49. Automatic Rule Application

The long-term BWL goal is:

entity Employee {
    id: auto
    name: required text
    email: required email
    age: number min 18
}

rule EmployeeDepartment {
    require Employee.department
}

Then:

form Employee

save Employee

table Employee editable

workflow EmployeeApproval

all understand the same validation model.

## 50. No Duplicate Validation Code

BWL should avoid this pattern:

form validation

plus:

save validation

plus:

API validation

plus:

workflow validation

plus:

database validation

when the same business rule is being repeated.

Instead:

entity Employee {
    age: number min 18
}

rule AdultEmployee {
    Employee.age >= 18
}

All relevant components reuse the same definitions.

## 51. Compiler Responsibilities

The BWL compiler should validate:

- Rule syntax
- Field references
- Business object references
- Type compatibility
- Rule dependencies
- Validation expressions
- Query expressions
- State references
- Workflow references
- Permission references

The compiler should report clear errors when rules are invalid.

## 52. Runtime Responsibilities

The BWL runtime should handle:

- Runtime validation
- Database-dependent rules
- Unique checks
- Business rule execution
- Validation errors
- Rule ordering
- Rule dependencies
- Transaction interaction
- Workflow validation
- Persistence validation

## 53. Error Model

Validation errors should have a consistent structure.

Conceptually:

ValidationError {
    object
    field
    rule
    message
}

For example:

ValidationError {
    object: Employee
    field: age
    rule: AdultEmployee
    message: "Employee must be at least 18 years old"
}

This common structure can be consumed by forms, APIs, tables, workflows, and interfaces.

## 54. User-Friendly Errors

Technical implementation details should not be required for normal users.

Instead of:

database constraint violation 23514

a BWL interface should be able to display:

Employee must be at least 18 years old.

The runtime may retain the technical error information for developers and logs.

## 55. Validation Flow

The complete validation flow is:

Business Object
    ↓
Basic Type Validation
    ↓
Field Constraints
    ↓
Business Rules
    ↓
Relationship Rules
    ↓
State Rules
    ↓
Authorization
    ↓
Persistence

The exact internal implementation may vary, but invalid business data must not be persisted.

## 56. Complete Example

entity Employee {
    id: auto
    name: required text min 2 max 100
    email: required unique email
    age: number min 18 max 100
    department: required text
    position: text
}

rule EmployeeManagerAge {
    if Employee.position = "Manager" and Employee.age < 21 {
        reject Employee with "Managers must be at least 21 years old"
    }
}

rule EmployeeDepartment {
    require Employee.department
}

Then:

form Employee

save Employee

The form automatically understands the field-level validation.

The save operation automatically applies the applicable business rules.

No duplicate validation code is required.

## 57. Complete Application Flow

A normal BWL application may therefore look like:

entity Employee {
    id: auto
    name: required text
    email: required unique email
    age: number min 18
    department: required text
}

rule EmployeeManagerAge {
    if Employee.position = "Manager" and Employee.age < 21 {
        reject Employee with "Managers must be at least 21 years old"
    }
}

form Employee

save Employee

table Employee

report Employee

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

All components reuse the same:

- Fields
- Types
- Validation
- Business rules
- Relationships
- Persistence model

## 58. Core Philosophy

BWL validation and rules follow the same fundamental principle:

> Define once. Reuse everywhere.

A business rule should not be copied into every screen.

A required field should not be redefined in every form.

A type constraint should not be manually repeated in every API.

A business invariant should not depend on one particular interface.

The business definition is the source of truth.

## 59. Summary

BWL provides a centralized validation and business-rule model.

The developer defines:

entity Employee {
    id: auto
    name: required text
    email: required unique email
    age: number min 18
    department: required text
}

and business rules such as:

rule EmployeeManagerAge {
    if Employee.position = "Manager" and Employee.age < 21 {
        reject Employee
    }
}

The same definitions are automatically reused by:

form Employee

save Employee

table Employee

report Employee

workflow EmployeeApproval

API Employee

The central rule remains:

> **One business definition. One validation model. Many automatic uses.**

The goal is:

> **Less code. Less duplication. Consistent business rules. More business meaning.**
