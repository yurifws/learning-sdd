# Java + Spring Boot — SDD Stack Guide

> Start here if you're building a Java backend with Spring Boot.
> This folder contains SDD applied to Java — stack-specific constitution, conventions, and AI onboarding.

---

## What's in This Folder

| File | What it does |
|---|---|
| [`CONSTITUTION.md`](CONSTITUTION.md) | Non-negotiable rules for this stack: architecture, naming, testing, migrations, security gates |
| [`AGENTS.md`](AGENTS.md) | AI onboarding file — full 6-layer code examples, naming conventions, safety rules |

---

## Reading Order

1. Read [`CONSTITUTION.md`](CONSTITUTION.md) first — understand the non-negotiable rules
2. Read [`AGENTS.md`](AGENTS.md) — this is what you paste into your AI session at the start of every task

---

## Stack

| Layer | Technology |
|---|---|
| Language | Java 21 (LTS) |
| Framework | Spring Boot 3.3.x |
| Architecture | Hexagonal (Ports & Adapters) or Layered (Controller → Service → Repository) |
| Database | PostgreSQL + Spring Data JPA |
| Migrations | Flyway |
| Testing | JUnit 5 + Testcontainers |
| Auth | Spring Security + JWT (RS256) |
| API Docs | SpringDoc OpenAPI 3 |

---

## Architecture Choice

This stack supports two architectures. Choose based on complexity:

| Architecture | Use when | Example |
|---|---|---|
| **Hexagonal** | Domain logic is complex, multiple adapters (REST + events + batch) | [`examples/projects/java-hexagonal/`](../../examples/projects/java-hexagonal/) |
| **Layered** | CRUD-heavy API, simple domain, small team | [`examples/projects/java-layered/`](../../examples/projects/java-layered/) |

---

## How to Use With the SDD Workflow

1. Copy `CONSTITUTION.md` into your project root — fill in the project-specific sections
2. Copy `AGENTS.md` into your project root — this becomes your AI's onboarding file
3. Follow the SDD workflow from [`sdd/README.md`](../../README.md):
   - Run the Clarification Gate
   - Write EARS requirements
   - Design
   - Break into atomic tasks
   - Feed the AI one task at a time
