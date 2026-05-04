## Monorepo 

The monorepo is the "housing" for your code. It solves the "Copy-Paste" problem by allowing shared code to live alongside applications.

Structure:

- `apps/`: Deployable units (e.g., Client App, Admin Portal).
- `packages/`: Shared libraries (UI Kit, logic hooks, shared types, network factories).

Key Benefit: Atomic Changes. You can update a shared TypeScript interface in packages/types and instantly see the red squiggly lines in every app that needs fixing.

Tooling: Turborepo is the engine that ensures you only build and test what has actually changed ("Affected" logic), keeping CI/CD fast.

## The Modular Monolith

Before splitting into multiple apps, you organize a single app into "Modules." This is a build-time isolation strategy.

- The Public API: Each module (e.g., `/billing`) has a single `index.ts`. Other modules can only import what is explicitly exported there.
- The Dependency Rule: Use `ESLint` or `Nx` rules to prevent "sideways" imports. `Feature A` should never import from `Feature B` directly; they should only talk via the `App Shell` or `Shared Packages`.
- Communication: Use Contracts (Events/Pub-Sub) or Component Injection (Slots). This ensures that if you delete one module, the rest of the app doesn't crash.

## Micro-Frontends (Module Federation)

When teams need to deploy independently, we move to runtime isolation using Webpack/Rspack Module Federation.

### Key Patterns:

- Bidirectional Federation: The Host provides the Layout/Auth context, while Remotes provide the specific page content.
- The Bridge (Legacy vs. Modern): If mixing versions (e.g., React 15 and React 19), you cannot use standard component nesting. You must use a Mount Function (the Host provides a DOM node, and the Remote "boots" its own React engine inside it).
- The Trailing Star: Use path="/invoice/*" in the Host's router to delegate all sub-routing to the Remote module.