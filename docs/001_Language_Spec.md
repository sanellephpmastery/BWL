# BWL Language Specification

## Introduction

BWL (Business Workflow Language) is a programming language designed to describe business processes in a simple and human-readable way.

## Core Concept

BWL separates business logic from software implementation.

The developer describes:

"What should the business do?"

The BWL compiler transforms it into:

"How should the software be built?"

## Basic Structure

A BWL program contains:

- Entities
- Workflows
- Rules
- Permissions
- Interfaces
- Reports

## Example

```bwl
workflow OrderProcessing {

    receive Order

    validate Payment

    if approved {
        ship Product
    }

}
