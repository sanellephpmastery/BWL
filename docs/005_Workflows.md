# BWL Workflows

## Purpose

This document defines workflows in BWL (Business Workflow Language).

A workflow describes a business process as a sequence of business actions, decisions, and outcomes.

A workflow focuses on **what the business process does**, rather than how software implements it.

## 1. Basic Workflow

A workflow is declared using the `workflow` keyword.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    ship Product
}
```

The statements inside the workflow describe the business process in execution order.

## 2. Workflow Actions

Actions represent business activities.

Examples:

```bwl
receive Order
validate Payment
ship Product
send Invoice
```

Actions should use business terminology that can be understood independently from implementation technology.

## 3. Workflow Sequence

Statements inside a workflow execute in sequence unless a condition or another control structure changes the flow.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    prepare Product

    ship Product

    send Invoice
}
```

The process moves from one action to the next.

## 4. Workflow Conditions

A workflow may make decisions using conditions.

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

The condition determines which business path is followed.

## 5. Workflow Inputs

A workflow may receive business data as input.

```bwl
workflow OrderProcessing {

    receive Order
}
```

The `Order` entity becomes available to the workflow after it is received.

## 6. Workflow Outputs

A workflow may produce a business result.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    ship Product

    complete Order
}
```

The final action represents the business outcome of the workflow.

## 7. Workflow State

A workflow may have a business state.

Example:

```bwl
workflow OrderProcessing {

    state pending

    receive Order

    state processing

    validate Payment

    state completed

    complete Order
}
```

Workflow states describe where the business process currently is.

## 8. Workflow Transitions

Actions may cause a workflow to move from one state to another.

```bwl
workflow OrderProcessing {

    state pending

    receive Order

    state processing

    validate Payment

    state completed
}
```

The workflow should only move between valid states.

## 9. Approval Steps

Workflows may require approval before continuing.

```bwl
workflow OrderProcessing {

    receive Order

    if Order.total > 10000 {
        require ManagerApproval
    }

    ship Product
}
```

The workflow cannot continue until the required approval is satisfied.

## 10. Permissions in Workflows

Actions may require specific permissions.

```bwl
workflow OrderProcessing {

    receive Order

    require permission ApproveOrder

    approve Order

    ship Product
}
```

The permission determines who may perform the protected business action.

## 11. Rules in Workflows

A workflow may trigger or depend on business rules.

```bwl
workflow OrderProcessing {

    receive Order

    validate Order

    apply OrderApproval

    if Order.approved {
        ship Product
    }
}
```

Rules define business decisions while workflows define the process in which those decisions occur.

## 12. Workflow Errors

A workflow may encounter a business failure.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    if Payment.status = "failed" {
        reject Order
    } else {
        ship Product
    }
}
```

Business failures should be explicitly represented rather than hidden inside implementation code.

## 13. Workflow Completion

A workflow should have a clear business outcome.

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    ship Product

    complete Order
}
```

A completed workflow represents a successful business process.

## 14. Workflow Cancellation

A workflow may be cancelled when the business process can no longer continue.

```bwl
workflow OrderProcessing {

    receive Order

    if Order.cancelled {
        cancel Order
    }

    validate Payment
}
```

Cancellation should produce a defined business outcome.

## 15. Workflow Repetition

A workflow may need to process multiple items.

```bwl
workflow OrderProcessing {

    receive Order

    for item in Order.items {
        prepare item
    }

    ship Order
}
```

The repetition structure allows a business action to be applied to each item.

## 16. Workflow Events

A workflow may react to a business event.

```bwl
workflow PaymentProcessing {

    when PaymentReceived {

        validate Payment

        update Order

        send Receipt
    }
}
```

Events allow workflows to begin or continue when something happens in the business environment.

## 17. Workflow Composition

A workflow may use another workflow when a business process contains a reusable subprocess.

```bwl
workflow OrderProcessing {

    receive Order

    run PaymentProcessing

    ship Product
}
```

This allows larger business processes to be composed from smaller workflows.

## 18. Workflow Variables

A workflow may maintain temporary values needed during execution.

```bwl
workflow OrderProcessing {

    receive Order

    total: money = Order.total

    if total > 10000 {
        require ManagerApproval
    }
}
```

Workflow variables exist only for the duration of the workflow unless explicitly stored in an entity.

## 19. Workflow Determinism

A workflow should produce predictable results when given the same business state and inputs.

The compiler and runtime should avoid ambiguous execution order.

For example:

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    ship Product
}
```

The order of these actions is explicit.

## 20. Workflow Validation

Before execution, the compiler should validate:

* Referenced entities exist
* Referenced actions are valid
* Conditions are boolean
* Required permissions exist
* Required rules exist
* Workflow states are valid
* Workflow transitions are valid
* Inputs are compatible with their expected types
* Outputs represent valid business results

## 21. Example

A complete workflow example:

```bwl
entity Order {
    customer: Customer
    total: money
    approved: boolean
}

workflow OrderProcessing {

    state pending

    receive Order

    state processing

    validate Payment

    if Payment.status = "failed" {
        reject Order
        state cancelled
    } else if Order.total > 10000 {
        require ManagerApproval

        if Order.approved {
            ship Product
            send Invoice
            state completed
        }
    } else {
        ship Product
        send Invoice
        state completed
    }
}
```

This workflow demonstrates:

* Business entities
* Sequential actions
* Conditions
* Approval
* Permissions
* Workflow states
* Business outcomes

## 22. Design Principle

BWL workflows should describe **business processes directly**.

A developer should be able to implement the workflow in any suitable technology without changing the business meaning of the BWL program.

The workflow is the business process definition.

The runtime is responsible for executing that definition.
