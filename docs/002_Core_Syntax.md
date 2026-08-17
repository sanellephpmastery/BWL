# BWL Core Syntax

## Purpose

This document defines the core syntax of BWL (Business Workflow Language).

The syntax is designed to be:

* Human-readable
* Simple to learn
* Explicit about business logic
* Independent from programming languages
* Easy for a compiler to parse

## 1. Entities

Entities represent the important business objects in a system.

```bwl
entity Customer {
    name
    email
}
```

An entity may contain fields that describe its data.

## 2. Workflows

Workflows describe business processes as a sequence of actions.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    if approved {
        ship Product
    }
}
```

A workflow should describe **what the business process does**, not how the underlying software implements it.

## 3. Rules

Rules define business conditions that must be followed.

```bwl
rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

Rules express business decisions independently from implementation code.

## 4. Permissions

Permissions define who is allowed to perform an action.

```bwl
permission ApproveOrder {
    role Manager
}
```

Permissions are used to control access to business operations.

## 5. Interfaces

Interfaces describe how users or external systems interact with the business process.

```bwl
interface OrderForm {

    input Customer
    input Product
    submit Order
}
```

The interface describes the business interaction without defining a specific UI technology.

## 6. Reports

Reports describe information that the business needs to view or analyze.

```bwl
report SalesSummary {

    show Order
    show Customer
    show Order.total
}
```

Reports describe the required business information without depending on a specific reporting system.

## 7. Conditions

Conditions allow a workflow or rule to make decisions.

```bwl
if approved {
    ship Product
}
```

Conditions may use values from entities, workflow state, or business rules.

## 8. Actions

Actions describe something that happens during a workflow.

Examples:

```bwl
receive Order
validate Payment
ship Product
send Invoice
```

Actions should use business terminology rather than implementation-specific terminology.

## 9. Naming

BWL names should use clear business terms.

Examples:

```bwl
Customer
Order
Payment
OrderProcessing
ApproveOrder
SalesSummary
```

Names should be descriptive and easy for a business user to understand.

## 10. Design Principle

BWL syntax must preserve the separation between **business intent** and **software implementation**.

A BWL program describes:

> What the business wants to happen.

The BWL compiler is responsible for determining:

> How the software should make it happen.

## 11. Minimal BWL Program

A minimal BWL program may combine entities, rules, permissions, and workflows.

```bwl
entity Order {
    customer
    total
}

permission ApproveOrder {
    role Manager
}

rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}

workflow OrderProcessing {

    receive Order

    validate Payment

    if approved {
        ship Product
    }
}
```

This syntax forms the foundation for the BWL compiler and future language specifications.
