---
name: software-design-patterns
description: Recognize, explain, recommend, and implement the 22 classic GoF software design patterns (creational, structural, behavioral). Use when the user asks about design patterns, wants to refactor code using a pattern, asks which pattern fits a problem, or mentions terms like Factory, Singleton, Observer, Strategy, Decorator, Adapter, Builder, Proxy, Command, Composite, Visitor, Iterator, Mediator, Memento, State, Template Method, Flyweight, Bridge, Facade, Prototype, Abstract Factory, Chain of Responsibility. Full pattern details are offline in the reference files — no web access needed.
---

# Software Design Patterns

Source: [refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)

**Full offline details for all 22 patterns are in the reference files — read the relevant one for problem/solution/structure/pseudocode/pros-cons/relations:**

- Creational (5): [creational-patterns.md](creational-patterns.md)
- Structural (7): [structural-patterns.md](structural-patterns.md)
- Behavioral (10): [behavioral-patterns.md](behavioral-patterns.md)

---

## Quick Lookup

### Creational — object creation mechanisms

| Pattern | One-line intent | Trigger words |
|---|---|---|
| **Factory Method** | Subclasses decide which class to instantiate | `new` scattered, product type unknown at compile time |
| **Abstract Factory** | Produce families of related objects | Product families, UI themes, multi-platform |
| **Builder** | Construct complex objects step by step | Telescoping constructor, many optional params |
| **Prototype** | Clone existing objects without knowing their class | Copy, clone, expensive initialization |
| **Singleton** | One instance with a global access point | Shared resource, DB connection, config, logger |

### Structural — assembling objects and classes

| Pattern | One-line intent | Trigger words |
|---|---|---|
| **Adapter** | Make incompatible interfaces work together | Third-party class, legacy interface, wrapper |
| **Bridge** | Split abstraction and implementation into independent hierarchies | Orthogonal dimensions, class explosion |
| **Composite** | Treat individual objects and composites uniformly | Tree structure, part-whole hierarchy |
| **Decorator** | Attach behaviors at runtime by wrapping | Optional features, stackable responsibilities, `final` class |
| **Facade** | Simplified interface to a complex subsystem | Complex library, reduce coupling |
| **Flyweight** | Share common state across many objects | Huge number of similar objects, memory |
| **Proxy** | Substitute that controls access to the real object | Lazy init, access control, caching, logging, remote |

### Behavioral — algorithms and responsibility assignment

| Pattern | One-line intent | Trigger words |
|---|---|---|
| **Chain of Responsibility** | Pass requests along a handler chain | Middleware, pipeline, multiple possible handlers |
| **Command** | Encapsulate a request as an object | Undo/redo, queue, schedule, button actions |
| **Iterator** | Traverse collection without exposing internals | for-each over custom structure |
| **Mediator** | Reduce coupling via a central coordinator | Chaotic dependencies, form elements interacting |
| **Memento** | Save/restore state without breaking encapsulation | Undo, snapshot, rollback, transaction |
| **Observer** | Notify subscribers about publisher state changes | Events, pub/sub, reactive, MVC |
| **State** | Object changes behavior when state changes | FSM, big switch on state, document workflow |
| **Strategy** | Family of interchangeable algorithms | Swap algorithm at runtime, eliminate conditionals |
| **Template Method** | Skeleton algorithm; subclasses fill in steps | Shared algorithm structure with varying details |
| **Visitor** | Separate algorithm from objects it operates on | Add operations to hierarchy without modifying it |

---

## Identifying the Right Pattern

1. **Creating objects?** → Creational
2. **Assembling/composing objects?** → Structural
3. **Communication / algorithm assignment?** → Behavioral

## Code Smell → Pattern Map

| Code smell | Suggested pattern(s) |
|---|---|
| `new ConcreteClass()` scattered everywhere | Factory Method, Abstract Factory |
| Long constructor with many optional params | Builder |
| Cloning an object with private fields | Prototype |
| Single shared resource needed globally | Singleton |
| Using a class with an incompatible interface | Adapter |
| Class explosion from two independent dimensions | Bridge |
| Part-whole hierarchy needing uniform treatment | Composite |
| Adding optional features without subclassing | Decorator |
| Complex subsystem needs a simple entry point | Facade |
| Millions of similar objects eating memory | Flyweight |
| Lazy init / access control / caching around an object | Proxy |
| Sequential processing steps / middleware | Chain of Responsibility |
| Undo/redo, queued operations | Command + Memento |
| Traversing a custom data structure | Iterator |
| Many objects tightly coupled to each other | Mediator |
| State-dependent behavior with big switch blocks | State |
| Multiple algorithm variants swappable at runtime | Strategy |
| Same algorithm skeleton, varying details | Template Method |
| New operations on a class hierarchy without modifying it | Visitor |
| Notify many dependents on state change | Observer |

## Common Pattern Combinations

- **Factory Method + Strategy** — factory creates the right strategy at runtime
- **Decorator + Composite** — decorators wrap individual components; composites group them
- **Observer + Mediator** — mediator is publisher; components are subscribers
- **Command + Memento** — commands perform operations; mementos snapshot state for undo
- **Template Method + Strategy** — template method defines skeleton; Strategy can swap a step
- **Iterator + Visitor** — iterate over a complex structure and execute typed operations
- **Builder + Composite** — build complex Composite trees step by step
- **Proxy + Factory Method** — factory decides whether to return the real object or a proxy
