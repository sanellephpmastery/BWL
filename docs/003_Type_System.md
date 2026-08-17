# BWL Type System

## Purpose

This document defines the type system of BWL (Business Workflow Language).

The type system describes the kinds of values that BWL can understand and use in business processes.

The goal is to keep types:

* Simple
* Predictable
* Business-oriented
* Easy to read
* Easy for a compiler to validate

## 1. Text

The `text` type represents human-readable text.

```bwl
entity Customer {
    name: text
    address: text
}
```

Example value:

```bwl
"John Smith"
```

## 2. Number

The `number` type represents numeric values.

```bwl
entity Product {
    quantity: number
}
```

Example values:

```bwl
10
250
99.50
```

Numbers may be used in calculations and comparisons.

## 3. Boolean

The `boolean` type represents a true or false value.

```bwl
entity Order {
    approved: boolean
}
```

Example values:

```bwl
true
false
```

## 4. Date

The `date` type represents a calendar date.

```bwl
entity Order {
    orderDate: date
}
```

Example value:

```bwl
2026-08-17
```

Dates may be used for business rules and workflow conditions.

## 5. Money

The `money` type represents monetary values.

```bwl
entity Order {
    total: money
}
```

Example value:

```bwl
1500.00
```

Money is a distinct type because financial values have different business requirements from ordinary numbers.

## 6. Email

The `email` type represents an email address.

```bwl
entity Customer {
    email: email
}
```

Example value:

```bwl
"customer@example.com"
```

The compiler should validate that values assigned to an `email` field follow a valid email format.

## 7. Entity

An entity type represents a reference to another BWL entity.

```bwl
entity Customer {
    name: text
}

entity Order {
    customer: Customer
}
```

An `Order` can therefore refer to a `Customer`.

## 8. List

A list represents multiple values of the same type.

```bwl
entity Order {
    items: list<Product>
}
```

A list may contain zero or more values.

Example:

```bwl
products: list<Product>
```

## 9. Optional Values

A field may be optional when a value is not always required.

```bwl
entity Customer {
    name: text
    phone: text?
}
```

The `?` indicates that the value may be absent.

## 10. Type Rules

BWL should prevent incompatible values from being assigned to a field.

For example:

```bwl
entity Customer {
    age: number
}
```

This is valid:

```bwl
age: 25
```

This is invalid:

```bwl
age: "twenty five"
```

The compiler should detect incompatible types before the workflow is executed.

## 11. Type Usage in Conditions

Types may be used in business conditions.

```bwl
if Order.total > 10000 {
    require ManagerApproval
}
```

The compiler should verify that the values being compared are compatible.

## 12. Type Usage in Rules

Business rules may use typed values.

```bwl
rule LargeOrder {

    if Order.total >= 10000 {
        require ManagerApproval
    }
}
```

The type system ensures that `Order.total` can correctly participate in the comparison.

## 13. Type Usage in Workflows

Workflows may operate on typed entities and values.

```bwl
workflow OrderProcessing {

    receive Order

    if Order.approved {
        ship Product
    }
}
```

The compiler should verify that `Order.approved` is a boolean value.

## 14. Core Types

The initial BWL type system contains these core types:

| Type      | Purpose                     |
| --------- | --------------------------- |
| `text`    | Human-readable text         |
| `number`  | Numeric values              |
| `boolean` | True or false               |
| `date`    | Calendar dates              |
| `money`   | Monetary values             |
| `email`   | Email addresses             |
| `Entity`  | Reference to another entity |
| `list<T>` | Collection of values        |
| `T?`      | Optional value              |

## 15. Design Principle

BWL types should represent **business meaning**, not implementation details.

For example:

```bwl
price: money
```

is preferable to:

```bwl
price: float
```

because `money` communicates the business meaning of the value.

The type system exists to make BWL programs easier for both humans and compilers to understand.

## 16. Type Safety

BWL should favor type safety.

Before a BWL program can be executed, the compiler should detect:

* Invalid type assignments
* Invalid comparisons
* Invalid operations
* Invalid entity references
* Invalid list element types
* Invalid required values

This allows business errors to be discovered before execution.

## 17. Example

A complete example using the initial type system:

```bwl
entity Customer {
    name: text
    email: email
}

entity Order {
    customer: Customer
    total: money
    approved: boolean
    orderDate: date
}

rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}

workflow OrderProcessing {

    receive Order

    if Order.approved {
        ship Product
    }
}
```

This example demonstrates how BWL types provide clear meaning to business data while remaining independent from a specific programming language or database.
