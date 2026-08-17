# BWL Expressions and Conditions

## Purpose

This document defines how BWL expresses business logic through values, comparisons, logical conditions, and conditional execution.

Expressions allow BWL to describe business decisions in a clear and readable way.

## 1. Expressions

An expression produces a value.

Examples:

```bwl
Order.total
Customer.name
Order.approved
```

Expressions may refer to:

* Entity fields
* Literal values
* Calculations
* Comparisons
* Logical conditions

## 2. Literal Values

BWL supports basic literal values.

```bwl
"John Smith"
100
99.50
true
false
2026-08-17
```

The type of a literal value is determined by its syntax.

## 3. Comparison Operators

BWL supports comparison operators for business decisions.

| Operator | Meaning               |
| -------- | --------------------- |
| `=`      | Equal                 |
| `!=`     | Not equal             |
| `>`      | Greater than          |
| `<`      | Less than             |
| `>=`     | Greater than or equal |
| `<=`     | Less than or equal    |

Example:

```bwl
if Order.total > 10000 {
    require ManagerApproval
}
```

## 4. Equality

The `=` operator checks whether two values are equal.

```bwl
if Order.status = "approved" {
    ship Product
}
```

## 5. Not Equal

The `!=` operator checks whether two values are different.

```bwl
if Order.status != "cancelled" {
    process Order
}
```

## 6. Logical AND

The `and` operator requires all conditions to be true.

```bwl
if Order.total > 10000 and Order.approved = true {
    ship Product
}
```

Both conditions must be satisfied.

## 7. Logical OR

The `or` operator requires at least one condition to be true.

```bwl
if Payment.method = "card" or Payment.method = "bank" {
    process Payment
}
```

## 8. Logical NOT

The `not` operator reverses a boolean condition.

```bwl
if not Order.approved {
    request Approval
}
```

## 9. Grouping Conditions

Parentheses may be used to make complex conditions explicit.

```bwl
if (Order.total > 10000 and Order.approved = true)
    or Customer.vip = true {
    ship Product
}
```

Parentheses should be used when they improve clarity or change evaluation order.

## 10. Arithmetic Expressions

BWL may perform basic arithmetic on compatible numeric values.

Supported operators:

| Operator | Meaning        |
| -------- | -------------- |
| `+`      | Addition       |
| `-`      | Subtraction    |
| `*`      | Multiplication |
| `/`      | Division       |

Example:

```bwl
Order.total = Product.price * Order.quantity
```

Arithmetic should only be performed on compatible numeric types.

## 11. Conditional Execution

The `if` statement executes a block when its condition is true.

```bwl
if Order.approved {
    ship Product
}
```

The condition must evaluate to a boolean value.

## 12. Else

The `else` block executes when the `if` condition is false.

```bwl
if Order.approved {
    ship Product
} else {
    request Approval
}
```

## 13. Else If

Multiple business conditions may be evaluated using `else if`.

```bwl
if Order.total > 50000 {
    require DirectorApproval
} else if Order.total > 10000 {
    require ManagerApproval
} else {
    approve Order
}
```

Conditions are evaluated from top to bottom.

## 14. Boolean Expressions

Boolean expressions produce either `true` or `false`.

Examples:

```bwl
Order.approved
Customer.active
Order.total > 10000
Payment.status = "paid"
```

Boolean expressions may be combined with `and`, `or`, and `not`.

## 15. Null and Optional Values

Optional values may not contain a value.

For optional fields, BWL should provide a way to test whether a value exists.

Example:

```bwl
if Customer.phone exists {
    send SMS
}
```

A missing optional value should not automatically cause a workflow failure.

## 16. Conditions in Rules

Rules use expressions to describe business requirements.

```bwl
rule LargeOrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

The condition determines when the rule applies.

## 17. Conditions in Workflows

Workflows use conditions to control business process execution.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    if Payment.status = "paid" {
        ship Product
    } else {
        request Payment
    }
}
```

## 18. Type Compatibility

Expressions must respect the BWL type system.

Valid:

```bwl
Order.total > 10000
```

Invalid:

```bwl
Order.customer > 10000
```

The compiler should detect invalid operations before execution.

## 19. Operator Precedence

BWL evaluates expressions using a defined precedence order.

From highest to lowest:

1. Parentheses
2. Arithmetic operations
3. Comparisons
4. `not`
5. `and`
6. `or`

Parentheses should be preferred when a business rule could otherwise be misunderstood.

## 20. Design Principle

BWL expressions should prioritize **business readability**.

For example:

```bwl
if Order.total > 10000 and Customer.vip = true {
    require ManagerApproval
}
```

should be understandable by a business user without requiring knowledge of a programming language.

## 21. Example

A complete example using expressions and conditions:

```bwl
entity Customer {
    name: text
    vip: boolean
}

entity Order {
    customer: Customer
    total: money
    approved: boolean
}

rule OrderApproval {

    if Order.total > 10000 and not Order.approved {
        require ManagerApproval
    }
}

workflow OrderProcessing {

    receive Order

    if Order.approved {
        ship Product
    } else {
        request Approval
    }
}
```

Expressions and conditions provide the decision-making foundation of BWL.

They allow business rules and workflows to describe **when something should happen and why**.
