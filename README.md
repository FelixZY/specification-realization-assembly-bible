# The SRA Bible

SRA (Specification - Realization - Assembly) is an architectural pattern for
software development created by Felix Zedén Yverås. It is an attempt to
incorporate modern best practices with existing thoughts on software
architecture, such as DDD and Hexagonal.

SRA relies heavily on dependency injection and is designed for both binaries and
libraries. It natively supports projects with multiple outputs, making
multi-platform development a breeze.

## Core Philosophy: Right by Design

The core tenet of SRA is "Right by Design, Hard to Misalign", a translation
based on the Swedish phrase _lätt att göra rätt_ (lit. _easy to do right_).

By hiding implementations behind strict module boundaries and defaulting to
private or internal visibility, the pattern makes sure the path of least
resistance for a developer is to follow the its ruleset. Any deviations will
cause a compilation error, making such changes clear and intentional.

## The Structure

SRA divides code into a clear, three-tier hierarchy where dependencies flow from
specification to assembly.

```
Specification (library modules) - "The What"
├── Models
├── Capabilities
└── Events

Realization (library modules) - "The How"
├── Handlers
├── Persistence (Sqlite, Postgres, etc.)
└── Presentation (Compose, GTK, QT, etc.)

Assembly (binary modules) "The Connection"
├── Web
├── CLI
├── Windows
├── MacOS
├── Linux (deb)
├── Linux (rpm)
├── Android
└── iOS
```

Dependencies have a clear direction from specification to assembly, making it
difficult to interact with irrelevant code.

```
┌──────────────────────────────────────────────────────────┐
│                      SPECIFICATION                       │
│  (Models • Capabilities • Events)  <──┐                  │
└───────────────────────────────────────│──────────────────┘
                                        │
┌───────────────────────────────────────┴──────────────────┐
│                       REALIZATION                        │
│  (Handlers • Persistence • UI)     <──┤ (Implements Spec)│
└───────────────────────────────────────│──────────────────┘
                                        │
┌───────────────────────────────────────┴──────────────────┐
│                        ASSEMBLY                          │
│  (CLI • Mobile • Web • Windows)       (Binds Spec+Real)  │
└──────────────────────────────────────────────────────────┘
```

### Specification - The What

The specification is the source of truth and heart of the system. This is the
home of application-wide models and contracts describing the capabilities of the
software.

The specification typically represents the public API of the software and is
completely agnostic to its inner workings. This makes major changes, such as
changing the database backend or even core logic, possible without major
refactoring across the project. The use of contracts additionally allows for
stronger test isolation as unit tests never need to rely on an actual
implementation.

### Realization - The How

The realization is where the magic happens. Typically split between
"Application" and "Infrastructure" in DDD, the realization contains
implementations of the contracts exposed by the specification. Note that most
code in the realization layer is generally private or internal.

The general logic of the application typically goes into the "Handlers"
submodule but effort should be taken to group code cleanly into minimal units.
Subdirectories like `Realization/Persistence/Sqlite` and
`Realization/Persistence/Postgres` can be a great way to achieve this!

Since the application capabilities live in the specification, the presentation
layer becomes a simple realization of the specification. Rather than a bully
that makes demands of the business logic - a common misstep when doing DDD - the
UI becomes a plugin that's easy to substitute or amend as needs grow.

### Assembly - The Connection

The assembly is where the final deliverable is hooked up and created. This could
be a library, an executable or a web bundle. Or all of them.

The sole role of the assembly layer is to connect the specified capabilities of
the application with the realized implementations using dependency injection.
This makes it simple to swap realizations based on the current environment or
output target (e.g. using `Sqlite` for the mobile application and `Postgres` for
the server application).

## Naming for Intent

Humans take the path of least resistance. Why use `ICar` when `Car` is right
there? SRA takes this into account and uses naming conventions to reinforce
architectural boundaries.

### Use `Car`, not `ICar`

SRA relies heavily on indirection and exposes the contracts in the specification
as its public API. The model/contract/interface is the "real" object as far as
the rest of the system is concerned - let's make it the real object in
developers' minds as well.

### Use `CarImpl`, not `Car`

In SRA, implementations are background details. Nine times out of ten,
developers should reach for the contract rather than the implementation. By
moving the `I` prefix on the contract to an `Impl` suffix on the implementation,
developers are steered toward the contract and only use the concrete
implementation intentionally.
