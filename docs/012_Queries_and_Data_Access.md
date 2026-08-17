# BWL Queries and Data Access

## Purpose

This document defines how BWL retrieves and accesses business data.

The central principle is:

> Ask for business data. Let BWL handle the data access.

BWL queries should work with business objects and business fields instead of requiring developers to repeatedly write database-specific query code.

The developer describes what data is needed.

The compiler and runtime determine how that data is retrieved.

## 1. Business Object as the Query Source

Queries operate on existing business objects.

For example:

entity Employee {
    id: auto
    name: required text
    email: required email
    age: number
    department: text
    position: text
}

A query can simply reference:

get Employee

The query automatically knows the fields defined by `Employee`.

The fields do not need to be declared again.

## 2. Basic Query

The simplest query is:

get Employee

This means:

> Retrieve Employee business records.

BWL determines the underlying data-access operation.

The developer does not need to manually specify the database table.

## 3. Selecting Specific Fields

A query may request only specific fields.

get Employee.name, Employee.email, Employee.department

This means:

> Retrieve only the requested Employee information.

The fields already exist in the Employee definition.

No duplicate field declaration is required.

## 4. Complete Object Retrieval

A complete business object may be retrieved with:

get Employee

The result represents the Employee business object.

If Employee later gains another field:

entity Employee {
    id: auto
    name: text
    email: email
    department: text
    position: text
}

the complete query automatically understands the updated structure.

## 5. Field References

Fields are referenced using the business-object notation:

Employee.name

Employee.email

Employee.department

Employee.position

This same notation is used consistently across:

- Queries
- Filters
- Reports
- Rules
- Workflows
- Tables
- Interfaces

## 6. Filtering

Queries may filter data.

get Employee where Employee.department = "IT"

This means:

> Retrieve Employees whose department is IT.

The filter uses the existing business field.

The developer does not need to know the underlying database column name.

## 7. Multiple Conditions

Multiple conditions may be combined.

get Employee where Employee.department = "IT" and Employee.age >= 18

The query uses the BWL expression system.

The compiler verifies that:

- Employee exists
- department exists
- age exists
- the comparison types are valid

## 8. OR Conditions

Queries may use OR conditions.

get Employee where Employee.department = "IT" or Employee.department = "HR"

This retrieves Employees belonging to either department.

## 9. Grouped Conditions

Complex conditions may be grouped.

get Employee where (Employee.department = "IT" or Employee.department = "HR") and Employee.age >= 18

Grouping determines the logical evaluation of the conditions.

The BWL compiler should preserve the intended business meaning when translating the query.

## 10. Comparison Operators

BWL queries may support common comparison operators:

=

!=

>

<

>=

<=

These operators should work according to the data type of the referenced field.

For example:

get Employee where Employee.age >= 18

is valid when `age` is numeric.

## 11. Text Matching

Text fields may support matching operations.

get Employee where Employee.name contains "John"

This means:

> Retrieve Employees whose name contains the specified text.

The underlying implementation is handled by the persistence provider.

## 12. Starts With

A text field may be queried using:

get Employee where Employee.name startsWith "John"

This retrieves values whose name begins with the specified text.

## 13. Ends With

A text field may be queried using:

get Employee where Employee.name endsWith "son"

This retrieves values whose name ends with the specified text.

## 14. Empty Values

Queries may check whether a value is empty or missing.

get Employee where Employee.department is empty

The runtime translates this into the appropriate persistence operation.

## 15. Non-Empty Values

The opposite condition may be expressed as:

get Employee where Employee.department is not empty

This retrieves Employees with a department value.

## 16. Null Values

BWL may support explicit null checks.

get Employee where Employee.department is null

and:

get Employee where Employee.department is not null

The compiler should distinguish null handling from normal text or numeric comparisons.

## 17. Sorting

Results may be sorted.

get Employee order by Employee.name

The default direction may be ascending.

Descending order may be specified:

get Employee order by Employee.name desc

## 18. Multiple Sorting Fields

Multiple fields may be used for sorting.

get Employee order by Employee.department, Employee.name

The query first sorts by department and then by name.

Different directions may be specified:

get Employee order by Employee.department asc, Employee.name desc

## 19. Limiting Results

A query may limit the number of returned records.

get Employee limit 20

This means:

> Return at most 20 Employees.

The persistence provider should perform the limit efficiently.

## 20. Pagination

Queries may use pagination.

get Employee page 1 size 50

This requests the first page with up to 50 records.

A later page may be:

get Employee page 2 size 50

The runtime handles the appropriate pagination mechanism.

## 21. Offset

Where required, a query may support an offset.

get Employee offset 100 limit 50

This requests records beginning after the first 100 results.

The preferred high-level BWL syntax should remain simple and database-independent.

## 22. Counting Records

A query may count records.

count Employee

A filtered count may be:

count Employee where Employee.department = "IT"

The result is a number.

## 23. Existence Checks

BWL may check whether matching records exist.

exists Employee where Employee.email = "user@example.com"

This returns a boolean result.

This is useful for validation and business rules.

## 24. Single Record

A query may request one matching record.

get one Employee where Employee.id = 1001

If no matching record exists, the runtime returns the appropriate not-found result.

## 25. First Record

A query may request the first matching record.

get first Employee order by Employee.name

The runtime determines the most efficient implementation.

## 26. Relationships

Queries may navigate business relationships.

For example:

entity Department {
    id: auto
    name: text
}

entity Employee {
    id: auto
    name: text
    department: Department
}

A query may filter using the related object:

get Employee where Employee.department.name = "IT"

The relationship was defined once in the business model.

The query reuses it.

## 27. Related Data

A query may request related information.

get Employee.name, Employee.department.name

The result may contain:

Employee name

Department name

No separate relationship mapping is required.

## 28. Querying Related Objects

Business objects may be queried independently.

get Department where Department.name = "IT"

The same Department object can then be referenced by other BWL features.

## 29. Nested Queries

BWL may support nested business queries where useful.

For example:

get Employee where Employee.department in (
    get Department.name where Department.name = "IT"
)

The exact syntax may evolve as the language specification becomes more formal.

The important principle is that nested queries operate on business objects rather than raw database tables.

## 30. Query Variables

Query results may be assigned to variables.

employees = get Employee where Employee.department = "IT"

The result can then be reused.

for employee in employees {
    notify employee.email
}

The variable contains business data understood by BWL.

## 31. Query Results as Business Objects

Query results should preserve business-object semantics.

For example:

employees = get Employee

The result should contain Employee objects or an appropriate collection representation.

The developer should not need to manually reconstruct Employee objects from database rows.

## 32. Collections

Multiple records may be returned as a collection.

employees = get Employee

The collection may be used by:

- Tables
- Reports
- Workflows
- Interfaces
- Business logic
- APIs

The same business-object structure is preserved.

## 33. Iteration

BWL may iterate over query results.

employees = get Employee

for employee in employees {
    notify employee.email
}

The developer works with `employee` as an Employee business object.

## 34. Query Reuse

A query may be defined for reuse.

query ActiveEmployees {
    get Employee where Employee.status = "active"
}

Other components may use:

use ActiveEmployees

This avoids duplicating frequently used query logic.

## 35. Named Queries

Named queries may provide a reusable business-level data-access definition.

query EmployeesByDepartment(departmentName) {
    get Employee where Employee.department = departmentName
}

The query may then be used as:

EmployeesByDepartment("IT")

This keeps common data-access logic centralized.

## 36. Query Parameters

Queries may accept parameters.

query EmployeesByAge(minAge) {
    get Employee where Employee.age >= minAge
}

Usage:

EmployeesByAge(18)

The parameter is part of the query definition.

## 37. Parameter Validation

Query parameters should respect their declared types.

query EmployeesByAge(minAge: number) {
    get Employee where Employee.age >= minAge
}

Passing incompatible data should be rejected.

The compiler may detect invalid parameter types when they are known at compile time.

## 38. Query Security

Queries must respect BWL permissions.

A user should not gain access to data simply because a query exists.

For example:

permission ViewEmployee {
    role HR
}

A query may require:

get Employee requires permission ViewEmployee

The runtime must enforce the authorization decision.

## 39. Row-Level Access

BWL may support data access rules that limit which records a user may retrieve.

For example:

rule EmployeeDepartmentAccess {
    Employee.department = currentUser.department
}

This means a user may only access Employees belonging to the permitted department.

The exact security syntax may evolve, but the query system must support business-level access restrictions.

## 40. Query and Validation

Queries should use valid business fields.

This is valid:

get Employee where Employee.age >= 18

This should fail:

get Employee where Employee.unknownField = 18

The compiler should report that `unknownField` does not exist.

## 41. Query and Type Safety

BWL should prevent incompatible comparisons.

For example:

entity Employee {
    age: number
}

This should be invalid:

get Employee where Employee.age = "hello"

because `age` is numeric.

Type safety reduces runtime errors.

## 42. Query and Persistence

Queries operate through the persistence abstraction.

The flow is:

BWL Query
    ↓
Compiler
    ↓
Persistence API
    ↓
Persistence Provider
    ↓
Database or External Source

The BWL business code remains independent from the underlying storage technology.

## 43. Database Independence

The developer should not need to write different queries for different databases.

For example:

get Employee where Employee.department = "IT"

should represent the same business request regardless of whether the provider uses:

- PostgreSQL
- MySQL
- SQLite
- Another supported provider

The provider translates the operation appropriately.

## 44. No Raw SQL for Normal Queries

BWL should avoid requiring raw SQL for ordinary business data access.

Instead of:

SELECT *
FROM employees
WHERE department = 'IT';

the developer writes:

get Employee where Employee.department = "IT"

This makes the business intent clear.

## 45. Raw Queries

Advanced applications may sometimes require provider-specific queries.

BWL may support an explicit escape hatch for this.

For example:

raw query EmployeeDatabase {
    ...
}

Raw access should be clearly distinguished from normal BWL queries.

It should not become necessary for ordinary business operations.

## 46. Query Optimization

The compiler/runtime may optimize queries.

For example:

get Employee.name, Employee.department
where Employee.department = "IT"

does not necessarily require retrieving unrelated Employee fields.

The runtime may request only the necessary data from the persistence provider.

## 47. Lazy Data Access

Where supported, related data may be loaded only when needed.

For example:

get Employee

may initially retrieve Employee data without loading every related object.

When the application accesses:

Employee.department

the runtime may retrieve the related data.

The exact behavior depends on the runtime and provider.

## 48. Eager Data Access

A query may request related information explicitly.

get Employee.name, Employee.department.name

This tells the runtime that the related Department information is required.

The provider may optimize the retrieval accordingly.

## 49. Query Caching

BWL runtimes may cache safe query results when configured.

For example:

cache query ActiveEmployees for 60 seconds

Caching must respect:

- Permissions
- Data freshness
- Business rules
- Transaction state

Caching should not expose data to unauthorized users.

## 50. Query Consistency

The runtime should provide predictable consistency behavior according to the configured persistence provider.

When a save occurs:

save Employee

a subsequent query:

get Employee

should observe the expected persisted state according to the application's consistency model.

## 51. Transactions and Queries

Queries may execute inside transactions.

transaction {
    employee = get Employee where Employee.id = 1001
    employee.department = "IT"
    save employee
}

The runtime manages the transaction boundaries.

## 52. Query and Business Rules

Queries may be used by business rules.

rule UniqueEmployeeEmail {
    if exists Employee where Employee.email = Employee.email {
        reject Employee
    }
}

The exact identity comparison semantics must prevent the current record from incorrectly matching itself during updates.

The runtime should provide appropriate business-level query behavior.

## 53. Query and Forms

A form may use queries to populate selectable data.

For example:

form Employee {
    department: select from Department
}

The form can retrieve Department options using the existing Department business object.

No manual database query is required.

## 54. Query and Tables

Tables may use queries.

table Employee
    where Employee.department = "IT"
    order by Employee.name

The table uses the existing query model.

## 55. Query and Reports

Reports may use the same query capabilities.

report Employee
    where Employee.department = "IT"
    order by Employee.name

The report does not require a separate query language.

## 56. Query and APIs

An API may expose business query results.

expose Employee

The API can use the same business-object access model.

For example:

API request
    ↓
Permission
    ↓
BWL Query
    ↓
Persistence
    ↓
Employee result

The API does not need to duplicate database access logic.

## 57. Query and Workflows

Workflows may query business objects.

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

A task may retrieve related data:

employee = get Employee where Employee.id = task.EmployeeId

The workflow operates on the business object.

## 58. Query and Events

Events may trigger queries.

on EmployeeCreated {
    employee = get Employee where Employee.id = event.EmployeeId
}

The event system and query system reuse the same business data model.

## 59. Query and Notifications

A notification may use query results.

employee = get Employee where Employee.id = 1001

notify employee.email

The developer does not manually reconstruct the Employee data.

## 60. Query and Data Access Boundaries

BWL should keep data access centralized.

The preferred structure is:

Business Definition
    ↓
Query
    ↓
Persistence API
    ↓
Provider

Application components should not bypass this structure unnecessarily.

## 61. Query Errors

Queries may produce errors such as:

- Invalid business object
- Invalid field
- Invalid parameter
- Type mismatch
- Permission denied
- Data source unavailable
- Record not found
- Provider error

BWL should expose these through a consistent error model.

## 62. Not Found

A single-record query may produce a not-found result.

get one Employee where Employee.id = 999999

If no Employee exists, BWL should return a defined not-found state rather than an ambiguous result.

## 63. Empty Collections

A collection query with no matches should return an empty collection.

get Employee where Employee.department = "Unknown"

The result should be:

[]

rather than an undefined or invalid collection.

## 64. Query Result Metadata

The runtime may provide metadata such as:

- Total count
- Current page
- Page size
- Has next page
- Has previous page

This is useful for tables, reports, and interfaces.

## 65. Aggregations

BWL may support business-level aggregations.

count Employee

sum Employee.salary

average Employee.salary

min Employee.salary

max Employee.salary

Aggregations should use existing business fields.

## 66. Grouping

Queries may group data.

get Employee
    group by Employee.department

Grouping may be combined with aggregation.

count Employee
    group by Employee.department

The result represents business-level grouped data.

## 67. Aggregation Filtering

Aggregated results may support conditions.

count Employee
    group by Employee.department
    where Employee.age >= 18

The query first applies the data filter and then performs the grouping.

The exact execution model may be optimized by the runtime.

## 68. Date and Time Queries

BWL should support date and time comparisons where the business object contains date/time fields.

entity Employee {
    id: auto
    name: text
    hiredAt: datetime
}

A query may be:

get Employee where Employee.hiredAt >= today

The runtime handles the appropriate date/time representation.

## 69. Current Date and Time

BWL may provide business-level values such as:

today

now

currentUser

These values allow queries to express business requirements without database-specific functions.

## 70. Query Expressions

Queries use the same expression model used elsewhere in BWL.

This provides consistency between:

- Rules
- Validation
- Queries
- Reports
- Workflows
- Conditions

A developer should not have to learn a completely different expression syntax for each subsystem.

## 71. Query Composition

Queries may be composed from reusable definitions.

query ActiveEmployees {
    get Employee where Employee.status = "active"
}

query ITEmployees {
    use ActiveEmployees
    where Employee.department = "IT"
}

This allows larger queries to build on smaller business-level definitions.

## 72. Reusable Data Access

The goal is to avoid repeating common data-access logic.

Instead of:

get Employee where Employee.status = "active"

appearing in many files, a reusable query may define:

query ActiveEmployees {
    get Employee where Employee.status = "active"
}

Other components can then reuse:

use ActiveEmployees

## 73. Query Documentation

Named queries should be self-describing.

For example:

query ActiveEmployees {
    get Employee where Employee.status = "active"
}

The query name communicates the business purpose.

This is preferable to hiding important business logic inside raw database code.

## 74. Query Testing

Queries should be testable independently.

Example:

test ActiveEmployees {
    result = use ActiveEmployees

    expect result.count >= 0
}

More detailed tests may verify filtering, sorting, permissions, and expected results.

## 75. Compiler Responsibilities

The BWL compiler should validate:

- Query syntax
- Business object references
- Field references
- Field types
- Operators
- Query parameters
- Relationship paths
- Aggregations
- Sorting
- Grouping
- Pagination syntax
- Query composition
- Permission references

Invalid queries should produce clear compiler errors whenever possible.

## 76. Runtime Responsibilities

The runtime should handle:

- Query execution
- Parameter binding
- Permission enforcement
- Provider communication
- Transactions
- Pagination
- Result mapping
- Error handling
- Query optimization
- Optional caching

The developer should not manually implement these for normal BWL data access.

## 77. Parameter Safety

Query parameters must be passed as structured values rather than concatenated into raw database commands.

For example:

query EmployeesByDepartment(departmentName: text) {
    get Employee where Employee.department = departmentName
}

The runtime is responsible for safe parameter handling.

This prevents query injection problems in normal BWL queries.

## 78. Data Access Security

Security must remain part of the data-access pipeline.

The normal flow is:

Request
    ↓
Authentication
    ↓
Authorization
    ↓
Business Query
    ↓
Persistence Provider
    ↓
Result Filtering
    ↓
Response

A query should never bypass BWL access policies.

## 79. Result Filtering

Even when a query is valid, the runtime may need to remove fields or records that the current user is not permitted to access.

For example:

permission ViewEmployeeEmail {
    role HR
}

A non-HR user may receive Employee data without the restricted email field.

The exact security model may be defined by the BWL security specification.

## 80. Field-Level Access

BWL may support field-level access policies.

For example:

permission ViewEmployeeSalary {
    role Manager
}

The runtime may prevent unauthorized users from reading:

Employee.salary

This is applied consistently across tables, reports, APIs, and queries.

## 81. Query Independence from Storage

A BWL query should describe business intent.

For example:

get Employee where Employee.department = "IT"

The query should not care whether Employee is stored in:

- A relational table
- A document store
- An external service
- Another supported persistence system

The provider translates the request.

## 82. External Provider Queries

If a business object is backed by an external provider, the same query model may be used.

entity Customer from CustomerAPI

Then:

get Customer where Customer.status = "active"

The provider decides how to retrieve the matching records.

## 83. Provider Capability

Not every provider will support every query feature.

If a provider cannot efficiently support an operation, the runtime should:

- Translate it when possible
- Perform safe fallback processing when appropriate
- Reject unsupported operations clearly when necessary

The business query syntax should remain stable.

## 84. Query Performance

The runtime should optimize:

- Field selection
- Filtering
- Sorting
- Pagination
- Grouping
- Aggregation
- Relationship loading

The developer should be able to express business intent without manually optimizing every database query.

## 85. Query Safety

BWL should prevent accidental full-data operations where appropriate.

For example:

delete Employee

may require explicit confirmation or a condition when large-scale deletion is possible.

The exact safeguards are defined by the persistence and security systems.

## 86. Query and Delete Operations

Deletion should support conditions.

delete Employee where Employee.status = "archived"

This allows business-level bulk operations.

Such operations must respect permissions, validation, and transaction rules.

## 87. Query and Update Operations

Updates may also use business-level conditions.

update Employee
    where Employee.department = "IT"
    set Employee.status = "active"

The runtime performs the appropriate persistence operation.

The syntax may evolve, but the principle is:

> Express the business operation without writing database-specific code.

## 88. Bulk Operations

BWL may support bulk business operations.

archive Employee where Employee.status = "inactive"

The runtime may optimize this operation using provider-specific capabilities.

Business rules and permissions must still be respected.

## 89. Query Lifecycle

The complete query lifecycle is:

BWL Query
    ↓
Parse
    ↓
Validate
    ↓
Authorize
    ↓
Plan
    ↓
Execute
    ↓
Map Result
    ↓
Return Business Data

Each stage belongs to the BWL compiler/runtime architecture.

## 90. Core Philosophy

BWL queries follow the same principle as the rest of the language:

> Define once. Reuse everywhere.

Queries should work with business objects.

The developer should describe:

> What data is needed.

BWL determines:

> How to retrieve it.

## 91. Complete Example

entity Department {
    id: auto
    name: required text
}

entity Employee {
    id: auto
    name: required text
    email: required email
    age: number
    department: Department
    status: text
}

query ActiveITEmployees {
    get Employee
        where Employee.status = "active"
        and Employee.department.name = "IT"
        order by Employee.name
}

The query can be reused by:

table ActiveITEmployees

report ActiveITEmployees

API ActiveITEmployees

workflow EmployeeApproval

The query definition does not need to be rewritten.

## 92. Complete Application Flow

A complete BWL application may use:

entity Employee {
    id: auto
    name: required text
    email: required email
    age: number min 18
    department: text
    status: text
}

form Employee

save Employee

query ActiveEmployees {
    get Employee where Employee.status = "active"
}

table ActiveEmployees

report ActiveEmployees

workflow EmployeeApproval {
    Tasker -> Reviewer -> MasterReviewer
}

All components reuse the same:

- Business object
- Fields
- Types
- Validation
- Persistence
- Queries
- Workflow data

## 93. No Duplicate Data Access Code

BWL should avoid requiring:

findEmployee()

findEmployees()

findActiveEmployees()

findITEmployees()

findEmployeeByDepartment()

when these are simply business-level queries.

Instead, BWL provides:

get Employee

get Employee where Employee.status = "active"

get Employee where Employee.department = "IT"

The query itself expresses the business requirement.

## 94. No Database Table Dependency

A developer should not normally write:

get employees_table

when the business object is:

get Employee

The business name is the stable abstraction.

The database representation may change without requiring changes to normal business queries.

## 95. Query as a Business Abstraction

A query is not merely a database statement.

A BWL query represents:

> A request for business information.

For example:

get Employee where Employee.department = "IT"

means:

> Find Employees belonging to the IT department.

The compiler and runtime translate that business request into the appropriate data-access implementation.

## 96. Final Principle

BWL data access must preserve the main language philosophy:

> One business definition. One query model. Many automatic uses.

The developer should not need to repeatedly implement:

- Database queries
- Object mapping
- Parameter handling
- Pagination
- Result mapping
- Connection handling
- Security checks
- Query optimization

BWL should handle these through the compiler, runtime, and persistence provider.

The ultimate goal is:

> Less code.
>
> Less repetition.
>
> Safer data access.
>
> More business meaning.
