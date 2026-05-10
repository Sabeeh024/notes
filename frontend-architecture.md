# Monorepo, Modular Monolith & Micro-Frontend Notes

---

# 1. Monorepo

A **Monorepo** is the shared "housing" for your codebase.

It solves the **copy-paste problem** by allowing shared code to live alongside applications.

---

## Structure

### `apps/`

Deployable applications such as:

- Client App
- Admin Portal

---

### `packages/`

Shared reusable libraries such as:

- UI Kit
- Shared Hooks
- Shared Types
- Network Factories

---

## Key Benefit: Atomic Changes

You can update a shared TypeScript interface inside:

- `packages/types`

And immediately see type errors across all affected apps.

This creates:

- Safer refactoring
- Better consistency
- Faster feedback loops

---

## Tooling

### Turborepo

Turborepo acts as the build orchestration engine.

### Main Advantage

Only builds/tests what actually changed.

Often called:

- "Affected" logic

### Benefits

- Faster CI/CD
- Better caching
- Improved developer productivity

---

# 2. The Modular Monolith

Before splitting into multiple applications, a single application can be organized into isolated **Modules**.

This is primarily a:

> Build-time isolation strategy

---

## Core Principles

### The Public API

Each module should expose a single public entry point.

Example:

- `index.ts`

Other modules should only import from this public API.

---

### The Dependency Rule

Prevent direct sideways imports between features.

Example:

- `Feature A` should not directly import from `Feature B`

Instead, communication should happen through:

- App Shell
- Shared Packages

---

## Enforcing Boundaries

Use tools like:

- `ESLint`
- `Nx`

To enforce architectural rules.

---

## Communication Patterns

### Contracts (Events / Pub-Sub)

Modules communicate through:

- Events
- Publish/Subscribe systems

---

### Component Injection (Slots)

Allows modules to inject UI/components into controlled extension points.

---

## Key Goal

A module should be removable without crashing the entire application.

This improves:

- Isolation
- Maintainability
- Scalability

---

# 3. Micro-Frontends (Module Federation)

When teams require **independent deployments**, applications move toward runtime isolation.

Commonly implemented using:

- Webpack Module Federation
- Rspack Module Federation

---

## Key Patterns

### Bidirectional Federation

#### Host Responsibilities

The Host application provides:

- Layout
- Authentication
- Shared Context

---

#### Remote Responsibilities

Remote applications provide:

- Feature pages
- Domain-specific functionality

---

## The Bridge (Legacy vs Modern)

When mixing different framework versions:

Example:

- React 15
- React 19

Standard component nesting becomes unsafe.

---

### Solution: Mount Function

Instead of nesting components directly:

1. Host provides a DOM node
2. Remote application mounts itself independently
3. Remote boots its own React runtime

This creates version isolation.

---

## The Trailing Star Pattern

Use wildcard routing delegation.

Example:

```tsx
path="/invoice/*"
```

### Purpose

Delegates all sub-routing responsibility to the remote module.

---

# 4. Architectural Evolution

Typical progression:

1. Modular Monolith
2. Monorepo
3. Micro-Frontends

---

## Trade-Off Summary

| Architecture     | Isolation Type   | Deployment Style        | Complexity | Best For               |
| ---------------- | ---------------- | ----------------------- | ---------- | ---------------------- |
| Modular Monolith | Build-time       | Single deployment       | Low        | Small/medium teams     |
| Monorepo         | Repository-level | Multiple apps           | Medium     | Shared ecosystems      |
| Micro-Frontends  | Runtime          | Independent deployments | High       | Large autonomous teams |

---

# 5. Key Takeaways

- **Monorepos** optimize code sharing and developer productivity
- **Modular Monoliths** enforce clean architectural boundaries
- **Micro-Frontends** optimize independent deployment and team autonomy
- Strong boundaries are more important than folder structure
- Tooling (`Turborepo`, `Nx`, `ESLint`) is critical for scalability
