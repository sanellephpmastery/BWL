# BWL Permissions

## Purpose

This document defines permissions and authorization in BWL (Business Workflow Language).

Permissions determine **who is allowed to perform a business action**.

BWL permissions should be expressed using business concepts such as roles, users, and authorized actions.

## 1. Basic Permission

A permission is declared using the `permission` keyword.

```bwl
permission ApproveOrder {
    role Manager
}
```

This permission allows users with the `Manager` role to perform the protected action.

## 2. Roles

Roles represent business responsibilities.

Examples:

```bwl
Manager
Employee
Accountant
Administrator
CustomerService
```

A role should describe a business responsibility rather than a technical implementation.

## 3. Permission Names

Permission names should clearly describe the action being protected.

Good examples:

```bwl
ApproveOrder
CancelOrder
RefundPayment
ViewCustomer
EditCustomer
ProcessPayment
```

Avoid vague permission names such as:

```bwl
DoSomething
AccessStuff
Permission1
```

## 4. Permission in Workflows

A workflow may require a permission before an action can be performed.

```bwl
workflow OrderProcessing {

    receive Order

    require permission ApproveOrder

    approve Order

    ship Product
}
```

The runtime must verify the permission before allowing the protected action.

## 5. Permission in Rules

Rules may require authorization.

```bwl
rule OrderCancellation {

    if Order.status = "active" {
        require permission CancelOrder
    }
}
```

The rule determines when the permission is required.

## 6. Multiple Roles

A permission may be available to more than one role.

```bwl
permission ApproveOrder {
    role Manager
    role Director
}
```

A user with either role may perform the protected action.

## 7. Role Hierarchy

BWL may support role inheritance when business responsibilities require it.

```bwl
role Director extends Manager
```

A `Director` may inherit permissions assigned to `Manager`.

Role inheritance should be explicit and should not be assumed automatically.

## 8. Permission Conditions

A permission may depend on a business condition.

```bwl
permission ApproveLargeOrder {

    role Manager

    if Order.total <= 50000
}
```

The permission is valid only when its condition is satisfied.

## 9. Resource Permissions

Permissions may be restricted to specific business resources.

```bwl
permission EditCustomer {

    role CustomerService

    resource Customer
}
```

This means the role may edit the specified business resource.

## 10. Action Permissions

Permissions protect specific business actions.

```bwl
permission RefundPayment {
    role Accountant
}
```

A workflow may then require the permission:

```bwl
workflow RefundProcessing {

    receive Payment

    require permission RefundPayment

    refund Payment
}
```

## 11. Permission Denial

When a user does not have the required permission, the action must not execute.

Example:

```bwl
workflow OrderProcessing {

    require permission ApproveOrder

    approve Order
}
```

If the actor lacks `ApproveOrder`, the `approve Order` action must be denied.

The runtime should return an explicit authorization failure.

## 12. Permission Evaluation

Before executing a protected action, the runtime should evaluate:

1. The current actor
2. The actor's roles
3. The required permission
4. Any permission conditions
5. The target business resource
6. Any applicable business rules

Only after authorization succeeds may the action execute.

## 13. Separation of Permission and Rule

Permissions and rules have different responsibilities.

A **permission** answers:

> Who is allowed to do this?

A **rule** answers:

> Under what business condition should this be allowed or required?

Example:

```bwl
permission ApproveOrder {
    role Manager
}

rule LargeOrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

The permission controls authorization while the rule controls the business requirement.

## 14. Permission Composition

A workflow may require more than one permission.

```bwl
workflow PaymentProcessing {

    require permission ProcessPayment
    require permission ApprovePayment

    process Payment
}
```

All required permissions must be satisfied unless an explicit alternative is defined.

## 15. Alternative Permissions

A business action may allow more than one authorized role.

```bwl
permission ApproveOrder {
    role Manager
    role Director
}
```

Either role may satisfy the permission.

## 16. Delegation

A permission may be temporarily delegated when a business process requires it.

```bwl
delegation ApproveOrder {

    from Manager
    to Assistant

    until 2026-12-31
}
```

Delegation must be:

* Explicit
* Time-limited when appropriate
* Authorized
* Auditable

The detailed delegation model may be expanded in a future authorization specification.

## 17. Permission Revocation

Permissions may be revoked.

```bwl
revoke permission ApproveOrder
from User
```

After revocation, the user must no longer be allowed to perform actions protected by that permission.

## 18. Permission Scope

Permissions should follow the principle of least privilege.

A role should receive only the permissions required to perform its business responsibilities.

For example:

```bwl
permission ViewCustomer {
    role CustomerService
}
```

does not automatically grant:

```bwl
EditCustomer
DeleteCustomer
```

Each protected action should have an explicit authorization requirement.

## 19. Permissions and Workflow States

Permissions may vary depending on workflow state.

```bwl
workflow OrderProcessing {

    state pending

    receive Order

    state approved

    require permission ShipOrder

    ship Product
}
```

The runtime should evaluate whether the action is permitted in the current workflow state.

## 20. Permissions and Business Rules

Permissions may work together with business rules.

```bwl
permission ApproveOrder {
    role Manager
}

rule LargeOrderApproval {

    if Order.total > 10000 {
        require ManagerApproval
    }
}
```

Both authorization and business requirements must be satisfied before the protected action proceeds.

## 21. Permission Validation

Before execution, the compiler should verify:

* Permission names are unique
* Referenced roles exist
* Referenced resources exist
* Permission conditions are valid
* Protected actions reference valid permissions
* Role inheritance does not create invalid cycles
* Permission dependencies are resolvable

## 22. Authorization Failure

Authorization failures should be explicit.

Example:

```bwl
if not permission ApproveOrder {
    deny OrderApproval
}
```

The runtime should prevent the unauthorized action and provide a meaningful business-level failure.

## 23. Example

A complete permissions example:

```bwl
role Manager
role Accountant
role CustomerService

permission ApproveOrder {
    role Manager
}

permission RefundPayment {
    role Accountant
}

permission EditCustomer {
    role CustomerService
}

workflow OrderProcessing {

    receive Order

    require permission ApproveOrder

    approve Order

    ship Product
}

workflow RefundProcessing {

    receive Payment

    require permission RefundPayment

    refund Payment
}
```

This example demonstrates:

* Business roles
* Action-specific permissions
* Workflow authorization
* Separation between roles and permissions

## 24. Design Principle

BWL permissions should make authorization **explicit, understandable, and auditable**.

The language should clearly separate:

* **Role** — who the actor is
* **Permission** — what the actor may do
* **Rule** — when a business requirement applies
* **Workflow** — how the business process proceeds

This separation allows BWL systems to enforce business authorization without coupling the language to a specific authentication or identity technology.
