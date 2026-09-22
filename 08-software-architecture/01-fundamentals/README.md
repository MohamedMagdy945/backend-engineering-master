# 01 — Fundamentals

Software architecture starts with understanding a few fundamental concepts that help us organize software, control complexity, and make systems easier to change and maintain.

This section focuses on the basic ideas behind good architectural decisions. These concepts will be used throughout the rest of the architecture roadmap.

## 01 — What is Architecture

Software architecture describes the high-level structure of a software system.

It is concerned with how the major parts of a system are organized, how they communicate, where responsibilities belong, and how dependencies are controlled.

Architecture is not only about folders, projects, or frameworks. It is mainly about making important structural decisions that affect how the system can evolve over time.

---

## 02 — Separation of Concerns

**Separation of Concerns (SoC)** means separating different responsibilities within a system.

Instead of having one component responsible for everything, responsibilities are divided into meaningful parts.

For example, an application may separate:

```text
Presentation
Business Logic
Data Access
Authentication
Validation
```

Each part focuses on its own responsibility instead of mixing unrelated concerns together.

Good separation makes code easier to understand, modify, and test.

---

## 03 — Coupling and Cohesion

**Coupling** describes how strongly different components depend on each other.

High coupling means that changing one component can easily affect many others.

**Cohesion** describes how closely related the responsibilities inside a component are.

A highly cohesive component focuses on a clear purpose.

A common goal is:

```text
Low Coupling
     +
High Cohesion
     ↓
Easier to Maintain
```

Understanding coupling and cohesion helps us decide how components should be divided and connected.

---

## 04 — Dependency Direction

**Dependency Direction** describes how dependencies flow between different parts of a system.

For example:

```text
Presentation
      ↓
Application
      ↓
Domain
```

A major goal is to prevent core business rules from becoming dependent on technical details such as databases, frameworks, or external services.

This concept becomes especially important when learning:

* Clean Architecture
* Hexagonal Architecture
* Onion Architecture
* Dependency Inversion

---

## How These Concepts Connect

These four concepts are closely related:

```text
What is Architecture?
          ↓
How should responsibilities be separated?
          ↓
Separation of Concerns
          ↓
How should components depend on each other?
          ↓
Coupling & Cohesion
          ↓
Which direction should dependencies point?
          ↓
Dependency Direction
```

Together, they provide the foundation for understanding more advanced architecture styles and patterns.

## Topics

```text
01 — What is Architecture
02 — Separation of Concerns
03 — Coupling and Cohesion
04 — Dependency Direction
```

After completing this section, move to:

```text
02 — Architecture Styles
```
