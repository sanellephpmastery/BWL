# BWL Interfaces

## Purpose

This document defines interfaces in BWL (Business Workflow Language).

Interfaces describe how users, systems, or external services interact with BWL business processes.

An interface describes **what can be provided, requested, submitted, or received** without defining the technical implementation.

## 1. Basic Interface

An interface is declared using the `interface` keyword.

```bwl id="q1a8nm"
interface OrderForm {

    input Customer
    input Product

    submit Order
}
```

The interface defines the business interaction required to create an order.

## 2. Inputs

Inputs represent information provided to a business process.

```bwl id="x4p7ks"
interface CustomerForm {

    input name: text
    input email: email
}
```

Inputs should use the BWL type system.

## 3. Outputs

Interfaces may define information returned by a business process.

```bwl id="m8c2vd"
interface OrderStatus {

    input Order

    output Order.status
    output Order.total
}
```

Outputs represent business information available to the caller.

## 4. Submit Actions

An interface may submit information to a workflow.

```bwl id="r6t9zw"
interface OrderForm {

    input Customer
    input Product

    submit Order
}
```

The `submit` action represents a business-level operation.

## 5. Interface Actions

Interfaces may expose business actions.

```bwl id="p3h7qa"
interface OrderManagement {

    action ApproveOrder
    action CancelOrder
    action ViewOrder
}
```

Each action should correspond to a meaningful business operation.

## 6. Interface and Workflows

An interface may start or interact with a workflow.

```bwl id="v9k2sx"
interface OrderForm {

    input Customer
    input Product

    submit Order
}

workflow OrderProcessing {

    receive Order

    validate Payment

    ship Product
}
```

The interface provides the business input while the workflow performs the business process.

## 7. Interface Permissions

Interface actions may require permissions.

```bwl id="f2m8rc"
interface OrderManagement {

    action ApproveOrder
        requires permission ApproveOrder

    action CancelOrder
        requires permission CancelOrder
}
```

The runtime must verify authorization before executing a protected action.

## 8. Interface Rules

An interface may be subject to business rules.

```bwl id="k7q1nd"
interface OrderForm {

    input Order

    validate Order

    submit Order
}
```

Validation rules should be evaluated before the business operation is completed.

## 9. Required Inputs

An input may be required.

```bwl id="c5w8yt"
interface CustomerForm {

    input name: text required
    input email: email required
    input phone: text
}
```

Required inputs must be provided before the operation can continue.

## 10. Optional Inputs

Optional inputs may be omitted.

```bwl id="b4r6px"
interface CustomerForm {

    input name: text required
    input phone: text?
}
```

An optional input does not prevent the interface operation from proceeding when absent.

## 11. Input Validation

Interfaces should validate inputs according to their declared types.

```bwl id="n8d3qm"
interface CustomerForm {

    input email: email required
}
```

The runtime should reject values that do not conform to the expected type.

## 12. Business Validation

Type validation is different from business validation.

Type validation:

```bwl id="j5k9vx"
input age: number
```

Business validation:

```bwl id="t3q7mb"
if Customer.age < 18 {
    reject Customer
}
```

Both forms of validation may be required.

## 13. Interface Events

An interface may expose business events.

```bwl id="z8p4cw"
interface PaymentEvents {

    receive PaymentReceived
    receive PaymentFailed
}
```

External systems may use events to communicate business occurrences.

## 14. External Systems

Interfaces may represent interactions with external systems.

```bwl id="u6n2ra"
interface PaymentGateway {

    send Payment

    receive PaymentResult
}
```

The interface describes the business-level exchange without specifying HTTP, REST, SOAP, or another technical protocol.

## 15. Interface Responses

An interface may return a business result.

```bwl id="e1m7qs"
interface OrderSubmission {

    input Order

    submit Order

    output Order.status
}
```

The response should represent meaningful business information.

## 16. Interface Errors

Business failures should be represented explicitly.

```bwl id="a9c5tr"
interface OrderSubmission {

    input Order

    submit Order

    on failure {
        return OrderRejected
    }
}
```

Errors should communicate business outcomes rather than expose unnecessary implementation details.

## 17. Interface States

An interface may expose the state of a workflow.

```bwl id="w2x8fk"
interface OrderStatus {

    input Order

    output Order.status
}
```

This allows users or external systems to observe the current business state.

## 18. Interface Security

Interfaces must respect BWL permissions.

An interface must not bypass authorization defined by workflows, rules, or permissions.

For example:

```bwl id="h6q3nv"
interface OrderManagement {

    action ApproveOrder
        requires permission ApproveOrder
}
```

The runtime must verify the actor before executing the action.

## 19. Interface Versioning

Interfaces may evolve over time.

A version may be declared when compatibility needs to be maintained.

```bwl id="s7m4qa"
interface OrderForm version 1 {

    input Customer
    input Product

    submit Order
}
```

Future versions may introduce new inputs or outputs without changing the meaning of the original version.

The detailed compatibility rules may be defined in a future specification.

## 20. Interface Composition

An interface may expose several related business operations.

```bwl id="d4p8yz"
interface OrderManagement {

    action CreateOrder
    action ViewOrder
    action ApproveOrder
    action CancelOrder
}
```

Each operation should have a clear business purpose.

## 21. Interface and Business Logic

Interfaces should not contain large amounts of business logic.

For example, the interface should expose:

```bwl id="q8r2xm"
action ApproveOrder
```

while the business logic remains in:

```bwl id="n5c7va"
rule OrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

This separation keeps the interface simple and reusable.

## 22. Interface Validation

Before execution, the compiler should verify:

* Interface names are unique
* Input names are valid
* Input types are valid
* Output references are valid
* Referenced workflows exist
* Referenced permissions exist
* Referenced rules exist
* Required inputs are correctly declared
* Interface actions are valid

## 23. Example

A complete interface example:

```bwl id="y6k3pt"
interface OrderManagement {

    input Customer
    input Product

    action CreateOrder
    action ViewOrder
    action ApproveOrder
    action CancelOrder

    action ApproveOrder
        requires permission ApproveOrder

    output Order.status
}

workflow OrderProcessing {

    receive Order

    validate Payment

    if Order.approved {
        ship Product
        complete Order
    }
}
```

The interface exposes the business operations while the workflow controls the actual business process.

## 24. Design Principle

BWL interfaces should describe **business interactions, not technical protocols**.

An interface answers:

> How can a user or external system interact with this business capability?

The interface should remain independent from implementation technologies such as:

* Web pages
* Mobile applications
* REST APIs
* GraphQL
* Databases
* Message queues

These technologies may implement the interface, but they should not define the business meaning of the interface.

## 25. Summary

Interfaces provide the boundary between BWL business processes and the outside world.

They define:

* Inputs
* Outputs
* Actions
* Events
* Validation
* Permissions
* Business responses
* Workflow interaction

The BWL runtime or compiler may later map these definitions to concrete technologies.
