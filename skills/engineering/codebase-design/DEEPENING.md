# Deepening

How to deepen a cluster of shallow modules safely, given its dependencies. Assumes the vocabulary in [SKILL.md](SKILL.md): **module**, **interface**, **seam**, **adapter**.

## Dependency categories

When assessing a candidate for deepening, use its dependencies to choose realistic tests. A dependency category alone does not justify restructuring.

### 1. In-process

Pure computation, in-memory state, no I/O. Combine cohesive computation when doing so simplifies callers and concentrates responsibility. Test it directly; in-process execution alone does not justify merging modules.

### 2. Local-substitutable

Dependencies with local test stand-ins (PGLite for Postgres, in-memory filesystem). Use a stand-in when it faithfully exercises the relevant contract. Its availability does not establish a good module boundary or prove production integration.

### 3. Remote but owned (Ports & Adapters)

Your own services across a network boundary (microservices, internal APIs). A **port** (interface) can isolate transport when it makes the owned logic easier to change or test. Use an in-memory **adapter** for focused tests and integration checks for the real transport when relevant.

Retain an existing clear boundary when another port would add only indirection.

### 4. True external (Mock)

Third-party services (Stripe, Twilio, etc.) you don't control. Substitute external responses through an existing seam or an injected dependency where useful. Keep the owned decisions real and preserve relevant error, retry and side-effect checks.

## Seam discipline

- **Justify the seam.** Name the variation, external dependency or testing difficulty it isolates. Adapter count alone does not establish its value.
- **Internal seams vs external seams.** A deep module can have internal seams (private to its implementation, used by its own tests) as well as the external seam at its interface. Don't expose internal seams through the interface just because tests use them.

## Testing strategy: replace, don't layer

- Before removing tests, map their material assertions to retained or replacement coverage. Delete redundant implementation-shaped tests only after the replacement exercises the relevant behaviour and failure paths.
- Prefer tests through the interface callers use. Keep focused internal tests when they expose failures more precisely.
- Assert observable outcomes, including relevant persisted state, authorization, errors, side effects and telemetry.
- Tests should usually survive internal refactors. When one changes, check whether it protects a real internal contract or merely mirrors the implementation.
