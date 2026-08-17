# BWL Rules

## Purpose

This document defines business rules in BWL (Business Workflow Language).

Rules describe business requirements, decisions, validations, and constraints that must be followed by a BWL system.

Rules answer the question:

> Under what business conditions must something happen?

## 1. Basic Rule

A rule is declared using the `rule` keyword.

```bwl id="8k7x4m"
rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

A rule contains a business condition and one or more consequences.

## 2. Rule Conditions

The condition determines when a rule applies.

```bwl id="j5s1pw"
rule LargeOrder {

    if Order.total >= 10000 {
        require ManagerApproval
    }
}
```

The condition must evaluate to a boolean value.

## 3. Rule Actions

A rule may require or trigger a business action.

```bwl id="8d9k3r"
rule PaymentValidation {

    if Payment.status != "paid" {
        request Payment
    }
}
```

Actions should describe business intent rather than implementation details.

## 4. Validation Rules

Rules may validate business data.

```bwl id="f0p6yb"
rule CustomerEmailRequired {

    if Customer.email = "" {
        reject Customer
    }
}
```

Validation rules prevent invalid business data from continuing through a process.

## 5. Required Conditions

A rule may require something before a process can continue.

```bwl id="n4c2tx"
rule LargeOrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

The required condition must be satisfied before the dependent action is allowed.

## 6. Approval Rules

Rules may require approval from a specific role.

```bwl id="4f8z2q"
rule DiscountApproval {

    if Order.discount > 20 {
        require ManagerApproval
    }
}
```

Approval requirements should be explicit and auditable.

## 7. Permission Rules

Rules may depend on permissions.

```bwl id="x7m1qa"
rule OrderCancellation {

    if Order.status = "active" {
        require permission CancelOrder
    }
}
```

The permission system determines whether the actor is authorized to perform the action.

## 8. Multiple Conditions

A rule may contain multiple conditions.

```bwl id="2p6v8c"
rule VIPOrderApproval {

    if Order.total > 10000 and Customer.vip = true {
        require ManagerApproval
    }
}
```

Logical operators follow the expression rules defined in `004_Expressions_and_Conditions.md`.

## 9. Multiple Outcomes

A rule may define different outcomes for different conditions.

```bwl id="k3d9s1"
rule OrderReview {

    if Order.total > 50000 {
        require DirectorApproval
    } else if Order.total > 10000 {
        require ManagerApproval
    } else {
        approve Order
    }
}
```

Conditions are evaluated in order.

## 10. Rule Scope

Rules should have a clear business scope.

Example:

```bwl id="w2r6hc"
rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

This rule applies to order approval decisions.

Rules should not contain unrelated business responsibilities.

## 11. Rule Naming

Rule names should clearly describe the business decision or requirement.

Good examples:

```bwl id="5j8p3x"
OrderApproval
PaymentValidation
CustomerEligibility
DiscountApproval
CreditLimit
```

Avoid vague names such as:

```bwl id="v7q1nm"
CheckSomething
DoStuff
ProcessRule
```

Clear names improve readability and maintainability.

## 12. Rule Priority

Some businesses may have rules that conflict or require a specific evaluation order.

BWL may support explicit priority.

```bwl id="r5t8w2"
rule EmergencyApproval priority 1 {

    if Order.emergency = true {
        require DirectorApproval
    }
}
```

A lower priority number represents a higher evaluation priority.

If priority is not specified, the compiler should use the default rule ordering defined by the runtime.

## 13. Rule Dependencies

A rule may depend on another rule.

```bwl id="c9x4p7"
rule PaymentValidation {

    if Payment.status != "paid" {
        reject Order
    }
}

rule OrderCompletion {

    requires PaymentValidation

    if Payment.status = "paid" {
        complete Order
    }
}
```

Dependencies make relationships between business rules explicit.

## 14. Rule Conflicts

Rules may produce conflicting outcomes.

For example:

```bwl id="n2k7vf"
rule DiscountApproval {

    if Order.discount > 20 {
        require ManagerApproval
    }
}

rule AutomaticDiscount {

    if Customer.vip = true {
        approve Discount
    }
}
```

If two rules produce incompatible outcomes, the compiler or runtime should detect the conflict rather than silently choosing an arbitrary result.

## 15. Rule Consistency

Rules should be deterministic whenever possible.

Given the same business state and inputs, a rule should produce the same result.

For example:

```bwl id="m6q2rt"
rule CreditLimit {

    if Customer.balance > Customer.creditLimit {
        block Order
    }
}
```

The result depends only on the defined business values.

## 16. Rule Execution

Rules may be evaluated:

* Before a workflow action
* During a workflow
* After a workflow action
* When a business event occurs
* When explicitly requested

Example:

```bwl id="z8c4pn"
workflow OrderProcessing {

    receive Order

    apply OrderApproval

    ship Product
}
```

The workflow controls when the rule is applied.

## 17. Rule Failure

When a rule cannot be satisfied, the business process should receive an explicit outcome.

Example:

```bwl id="q4v9sd"
rule PaymentRequired {

    if Payment.status != "paid" {
        reject Order
    }
}
```

A rule failure should not be silently ignored.

## 18. Rule Overrides

BWL may allow a rule to define an explicit override mechanism.

```bwl id="t6k1rx"
rule CreditLimit {

    if Customer.balance > Customer.creditLimit {
        block Order
    }
}
```

Any override should require explicit authorization and should be auditable.

The exact override mechanism will be defined by the permission and audit specifications.

## 19. Rule Evaluation

The BWL runtime should evaluate rules according to:

1. Rule dependencies
2. Rule priority
3. Business conditions
4. Required permissions
5. Required approvals
6. Rule outcomes

The runtime must not ignore a rule that is applicable to the current business state.

## 20. Rule Validation

Before execution, the compiler should verify:

* The rule has a valid name
* Conditions are boolean
* Referenced entities exist
* Referenced fields exist
* Referenced permissions exist
* Referenced rules exist
* Actions are valid
* Rule dependencies are resolvable
* Rule priorities are valid
* Expressions use compatible types

## 21. Example

A complete rules example:

```bwl id="u8s2ka"
entity Customer {
    name: text
    vip: boolean
    balance: money
    creditLimit: money
}

entity Order {
    customer: Customer
    total: money
    discount: number
}

rule CreditLimit {

    if Customer.balance > Customer.creditLimit {
        block Order
    }
}

rule DiscountApproval {

    if Order.discount > 20 {
        require ManagerApproval
    }
}

rule LargeOrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}

rule VIPOrder {

    if Customer.vip = true and Order.total > 5000 {
        approve Order
    }
}
```

These rules demonstrate how BWL can express:

* Validation
* Authorization
* Approval
* Business constraints
* Conditional decisions
* Customer-specific policies

## 22. Design Principle

BWL rules should represent **business policy independently from workflow implementation**.

A workflow describes:

> What process happens.

A rule describes:

> What business condition must be satisfied.

Keeping these responsibilities separate makes BWL easier to understand, validate, test, and implement.

Rules are a core part of the BWL business logic model.
