# test-manager-library-3145555-docs

> **ADSO 3145555 technical challenge**
> Schedule: **Wednesday, October 14, 2026, 6:50 a.m.–11:00 a.m. Colombia time.** Submission deadline: **11:00 a.m.**

This repository contains the Manager Library product specification and its Python and .NET technical references. The target implementation stack for the challenge is React and Laravel.

## Instructions

Each participant should fork all six project repositories and complete the challenge using the target specification.

1. Fork each repository listed below to your GitHub account.
2. Read [`spec-laravel/`](spec-laravel/) as the target specification for the React and Laravel implementation.
3. Use [`spec-python/`](spec-python/) and [`spec-.net/`](spec-.net/) as references for the same product's requirements and prior technical decisions. Do not treat their framework-specific choices as Laravel requirements.
4. Develop the solution in your forks.

## Target technology

| Layer | Challenge technology |
|---|---|
| Frontend | React |
| Backend | PHP with Laravel |

Product behavior—including user stories, acceptance criteria, business rules, API contract, and data model—must remain consistent with the target specification. Framework-specific implementation guidance is documented in `spec-laravel/`.

## Challenge purpose

The primary goal is to demonstrate Spec-Driven Development (SDD): reading, interpreting, and applying a specification to build a product in the required stack. Code is the means, not the end.

## Submission

Submit the work pushed to your forks before **11:00 a.m. Colombia time**. Work submitted after the deadline will not be reviewed. There is no extension; push partial progress with small commits during the challenge.

## Specification folders

| Folder | Purpose |
|---|---|
| [`spec-laravel/`](spec-laravel/) | Target specification for the React and Laravel challenge |
| [`spec-python/`](spec-python/) | Python technical reference for the same product |
| [`spec-.net/`](spec-.net/) | .NET technical reference for the same product |

All three folders describe the same product scope. The Laravel specification is the implementation target; framework-specific decisions from the Python and .NET references are not automatically part of that target.

## Reading the Laravel specification

Start with [`spec-laravel/README.md`](spec-laravel/README.md), which defines the reading order and ownership of each document. It covers the constitution, product specification, data model, API contract, architecture, technical plan, tasks, and ADRs.

## The six project repositories

| Repository | Responsibility |
|---|---|
| [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs) | Product specification and Python/.NET technical references |
| [`test-manager-library-3145555-api`](https://github.com/code-sena/test-manager-library-3145555-api) | PHP backend using Laravel |
| [`test-manager-library-3145555-app`](https://github.com/code-sena/test-manager-library-3145555-app) | React frontend |
| [`test-manager-library-3145555-page`](https://github.com/code-sena/test-manager-library-3145555-page) | Static public presentation site |
| [`test-manager-library-3145555-infra`](https://github.com/code-sena/test-manager-library-3145555-infra) | Containers, network, volumes, and an empty database service |
| [`test-manager-library-3145555-tool`](https://github.com/code-sena/test-manager-library-3145555-tool) | Demo-data seeding utility |
