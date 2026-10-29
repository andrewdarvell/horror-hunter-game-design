# ADR-001: Hexagonal architecture (Ports & Adapters)

- Status: Accepted
- Project: Horror Hunter
- Date: 2026-10-30

## Context

Horror Hunter is a turn-based, top-down roguelike built with LibGDX. The LibGDX project generator provides a module named `core`, which is part of the LibGDX application layer. We want game rules to remain testable and runnable without LibGDX, rendering, or platform dependencies.

## Decision

Use **hexagonal architecture (Ports & Adapters)** with dependencies pointing inward:

- `domain` is a pure Java module containing the game model, turn-based simulation, rules and domain-owned **ports** (interfaces).
- The generated LibGDX `core` is an application/presentation layer: input, rendering, screens, integration and composition.
- **Adapters** implement domain ports by using external libraries or platform APIs (e.g. SquidSquad for map generation).
- Desktop and possible Android launchers depend on the LibGDX application module; domain does not depend on them.
- A concrete adapter depends on `domain` and its external library, **never vice versa**.
- `domain` public APIs use project-owned types, not LibGDX or third-party types.
- Third-party libraries confined to presentation/build tooling (e.g. Box2D Lights, TexturePacker) do **not** need a domain port.
- Introduce a port when the domain needs a capability supplied from outside; do not wrap every library automatically.

Illustrative dependency direction:

```text
lwjgl3 / android
        |
        v
  libGDX core --------> external adapters
        |                    |
        +-------> domain <---+
```

Example: `domain` defines `MapGenerator.generate(seed): GameMap`, where `GameMap` is a domain model. A SquidSquad-based adapter implements the port and converts library types into `GameMap`.

## Consequences

- Domain logic can be tested with JUnit and fake/in-memory port implementations.
- External algorithms and libraries can be swapped without changing the domain contracts.
- Integration and type-mapping code belongs at the boundary, not inside the domain.
- The specific Gradle module/package layout for adapters will be decided after inspecting the generated project; this ADR fixes dependency direction, **not** a premature filesystem layout.

## Related decisions

- Desktop is the initial target; Android is optional.
- Choose the generated LibGDX **Game** template.
- The generator's `core` name stays intact; our separate pure-Java module is called `domain`.
