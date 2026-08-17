# BWL Reports

## Purpose

This document defines reports in BWL (Business Workflow Language).

Reports provide a way to present business information using existing BWL business objects.

The central principle of BWL is:

> Define business data once. Reuse it everywhere.

A report should not require the developer to redefine fields that BWL already knows.

## 1. Basic Report

A report may reference an existing business object.

```bwl
report Employee
```

This means:

> Generate a report using the available Employee data.

The report automatically knows the fields defined by `Employee`.

## 2. Complete Business Object

If an object contains many fields:

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

The report can simply reference it:

```bwl
report SchoolForm
```

The developer does not need to repeat all fields.

## 3. Selected Fields

A report may select specific fields.

```bwl
report SchoolForm.firstName, SchoolForm.lastName, SchoolForm.school
```

Only those fields are included in the report.

The field definitions still come from `SchoolForm`.

## 4. Field References

The notation:

```bwl
SchoolForm.firstName
```

references the `firstName` field belonging to `SchoolForm`.

Examples:

```bwl
SchoolForm.lastName
SchoolForm.age
SchoolForm.location
SchoolForm.school
```

The same field-reference syntax should be used consistently throughout BWL.

## 5. Automatic Report Structure

When a report references a complete object:

```bwl
report Employee
```

BWL determines the report structure from the existing definition.

For example:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
    position: text
}
```

The report automatically knows about:

```text
id
name
email
department
position
```

No duplicate declaration is required.

## 6. Report and Entity

Entities are the source of truth for reusable business data.

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
}
```

A report may reuse it:

```bwl
report Employee
```

If the entity changes, the report can automatically recognize the updated structure.

## 7. Report and Forms

A report may also reference a form-based business object.

```bwl
form SchoolForm {
    firstName: text
    lastName: text
    age: number
    school: text
}
```

Then:

```bwl
report SchoolForm
```

The report uses the existing form structure.

The fields are not declared a second time.

## 8. Filtering

Reports may filter business data.

```bwl
report Employee
    where Employee.department = "IT"
```

This means:

> Include only Employees whose department is `IT`.

The filter uses existing business fields.

## 9. Sorting

Reports may sort data.

```bwl
report Employee
    order by Employee.name
```

The report uses the existing `Employee.name` field.

## 10. Multiple Sorting Fields

A report may sort using multiple fields.

```bwl
report Employee
    order by Employee.department, Employee.name
```

The business fields remain defined only once.

## 11. Report Calculations

Reports may calculate values from existing fields.

For example:

```bwl
report Employee {
    Employee.name
    Employee.salary
}
```

A future calculation syntax may provide:

```bwl
Employee.salary * 12
```

The calculation uses the existing field rather than redefining it.

## 12. Report Grouping

Reports may group related business data.

Example:

```bwl
report Employee
    group by Employee.department
```

This allows a report to organize employees by department.

The grouping field comes from the existing entity.

## 13. Report Totals

Reports may calculate totals.

For example:

```bwl
report Employee {
    Employee.department
    total Employee.salary
}
```

The report uses the existing `salary` field.

No duplicate field definition is necessary.

## 14. Report Conditions

Reports may use existing BWL conditions.

```bwl
report Employee
    where Employee.age >= 18
```

The condition uses the same expression system used by rules and workflows.

## 15. Report Permissions

Reports must respect BWL permissions.

For example:

```bwl
permission ViewEmployeeReport {
    role Manager
}
```

The report may require that permission:

```bwl
report Employee
    requires permission ViewEmployeeReport
```

The report should not implement its own separate authorization system.

## 16. Report Output

A report represents business information.

The same report definition may later be rendered as:

- Web page
- Table
- PDF
- Spreadsheet
- Dashboard
- API response
- Other supported output

The BWL report definition should describe the business information rather than a specific presentation technology.

## 17. Report Presentation

Presentation options may be added without changing the underlying business data.

For example:

```bwl
report Employee
    order by Employee.name
```

The report describes the data.

The BWL compiler or runtime may determine how that report is rendered for the selected target.

## 18. Report Reuse

A report can be referenced by other parts of the system.

For example:

```bwl
report EmployeeSummary
```

A dashboard may use:

```bwl
use EmployeeSummary
```

An API may expose:

```bwl
expose EmployeeSummary
```

The exact implementation may vary, but the report definition remains the source of truth.

## 19. Report and Workflow

Reports may expose workflow information.

For example:

```bwl
report SchoolApproval
```

may use the workflow's existing business data and state.

A workflow remains responsible for process execution.

The report is responsible for presenting the business information.

## 20. Workflow Status

A report may display workflow status.

For example:

```bwl
report SchoolForm {
    SchoolForm.firstName
    SchoolForm.lastName
    SchoolForm.school
    workflow.status
}
```

The report can combine existing business data with workflow information without redefining the underlying data.

## 21. Automatic Updates

If the underlying business object changes:

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
report Employee
```

automatically has access to the new `position` field.

The developer does not need to manually update every report that uses the complete object.

## 22. Avoiding Repetition

BWL should avoid this:

```bwl
report Employee {
    id: number
    name: text
    email: email
    department: text
}
```

when the entity already exists:

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
}
```

Instead:

```bwl
report Employee
```

The report should reuse the existing definition.

## 23. Complete Example

```bwl
entity Employee {
    id: number
    name: text
    email: email
    department: text
    position: text
}

report Employee
```

This creates a report using the complete Employee structure.

A selected report may be:

```bwl
report Employee.name, Employee.email, Employee.department
```

A filtered report:

```bwl
report Employee
    where Employee.department = "IT"
```

A sorted report:

```bwl
report Employee
    order by Employee.name
```

## 24. School Example

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

Complete report:

```bwl
report SchoolForm
```

Selected report:

```bwl
report SchoolForm.firstName, SchoolForm.lastName, SchoolForm.school
```

Filtered report:

```bwl
report SchoolForm
    where SchoolForm.age >= 18
```

The fields are never redefined.

## 25. Report and Database

A report may read business data from the configured persistence layer.

For example:

```bwl
report Employee
```

does not require the developer to write:

```sql
SELECT ...
FROM employees
```

The BWL runtime/compiler handles the underlying data retrieval.

The developer describes the business information required.

## 26. Database Independence

BWL reports should remain independent of database technology.

A report should not need to know whether the data comes from:

- PostgreSQL
- MySQL
- SQLite
- MongoDB
- Another database
- An external service

The BWL business definition remains the same.

## 27. Report Validation

The compiler should verify:

- The referenced business object exists
- Referenced fields exist
- Filters use valid fields
- Sorting fields exist
- Grouping fields exist
- Calculations use valid expressions
- Required permissions exist
- Referenced workflow information exists

## 28. Design Principle

BWL reports follow the same principle as forms, tables, and persistence:

> **Define once. Reuse everywhere.**

A report should not become another place where the developer must redefine business data.

The report describes:

> What business information should be presented.

The compiler/runtime handles:

> How that information is retrieved and rendered.

## 29. BWL Ecosystem

Reports are part of the larger BWL ecosystem:

```text
Business Definition
       │
       ├── Form
       ├── Save
       ├── Table
       ├── Report
       ├── Workflow
       └── Interface
```

All of these should reuse the same underlying business definitions.

## 30. Core Philosophy

BWL should allow:

```bwl
form SchoolForm {
    ...fields...
}

save SchoolForm

table SchoolForm

report SchoolForm

workflow SchoolApproval {
    Tasker -> Reviewer -> MasterReviewer
}
```

Instead of repeatedly implementing:

- Field mappings
- Database operations
- Table columns
- Report fields
- Validation
- Workflow transitions

The language describes the business intent once, and the compiler/runtime handles the repetitive implementation.

## 31. Summary

BWL reports provide business reporting without duplicating business definitions.

The core rule is:

> **The business object is defined once.**

Then it may be reused as:

```bwl
form SchoolForm
save SchoolForm
table SchoolForm
report SchoolForm
```

Specific fields may be referenced with:

```bwl
SchoolForm.firstName
SchoolForm.lastName
SchoolForm.school
```

The BWL compiler and runtime are responsible for turning these declarations into the required application behavior.

The goal is simple:

> **Less code. Less repetition. More business meaning.**
