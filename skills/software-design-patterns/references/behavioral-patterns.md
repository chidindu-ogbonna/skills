# Behavioral Patterns — Full Reference

---

## Chain of Responsibility

*Also known as: CoR, Chain of Command*

**Intent:** Pass requests along a chain of handlers; each handler decides to process or forward.

**Problem:** A request must pass through multiple processing steps (auth check, validation, caching, rate limiting). Hardcoding the sequence couples all checks together and makes partial reuse impossible.

**Solution:** Extract each check into a handler class with a common interface. Link handlers in a chain; each handler either processes the request or passes it to the next. The chain can be assembled at runtime.

**Structure:**

- `Handler` interface — `handle(request)` + optional `setNext(handler)`
- `BaseHandler` — stores `next` reference; default `handle()` forwards to `next`
- `ConcreteHandler` — processes or forwards the request
- `Client` — assembles and triggers the chain

**Key pseudocode:**

```
abstract class Handler:
    protected next: Handler
    setNext(h): next = h; return h
    handle(req):
        if next != null: next.handle(req)

class AuthHandler extends Handler:
    handle(req):
        if not authenticated: deny
        else: super.handle(req)   // forward
```

**Applicability:**

- Multiple objects may handle a request; handler unknown at compile time
- Execute handlers in a specific, configurable order
- Set of handlers changes at runtime

**Pros:** Control over request handling order; SRP; OCP.
**Cons:** Some requests may go unhandled.

**Related:** CoR, Command, Mediator, and Observer all address sender-receiver connections. CoR is often used with Composite (bubble up the tree). CoR handlers can be Commands.

---

## Command

*Also known as: Action, Transaction*

**Intent:** Encapsulate a request as a stand-alone object, enabling parameterization, queuing, logging, and undo.

**Problem:** GUI elements (button, menu item, shortcut) need to trigger the same operations. Hardcoding click handlers creates duplicate logic and tight coupling between UI and business logic.

**Solution:** Extract each operation into a `Command` class with a single `execute()` method. UI elements store a command reference and call `execute()`. Commands encapsulate the receiver and all required parameters.

**Structure:**

- `Command` interface — `execute()`
- `ConcreteCommand` — implements `execute()`; holds `receiver` reference and parameters
- `Receiver` — contains the actual business logic
- `Invoker` — stores and fires commands; doesn't know the receiver
- `Client` — creates commands, wires invoker ↔ command ↔ receiver

**Key pseudocode:**

```
abstract class Command:
    method saveBackup(): backup = editor.text
    method undo(): editor.text = backup
    abstract execute()

class CutCommand extends Command:
    execute():
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true   // should be added to history
```

**Applicability:**

- Parameterize objects with operations
- Queue, schedule, or execute operations remotely
- Implement undo/redo

**Pros:** SRP; OCP; undo/redo; deferred/queued execution; compose commands.
**Cons:** Added layer between senders and receivers.

**Related:** CoR/Command/Mediator/Observer all connect senders and receivers. Use with Memento for undo: Command performs operation, Memento snapshots state. Command vs Strategy: Command converts an operation into an object (history, queuing); Strategy describes different ways to do the same thing. Prototype can save copies of Commands into history.

---

## Iterator

**Intent:** Traverse a collection without exposing its internal structure.

**Problem:** Different collection types (list, tree, graph) require different traversal logic. Embedding traversal in the collection blurs its responsibility; client code becomes coupled to specific collection internals.

**Solution:** Extract traversal into an `Iterator` object. The iterator tracks the current position and implements `hasMore()` / `getNext()`. Collections return iterators via a factory method.

**Structure:**

- `Iterator` interface — `hasMore(): bool`, `getNext(): T`
- `ConcreteIterator` — stores current position; linked to a specific collection
- `IterableCollection` interface — `createIterator(): Iterator`
- `ConcreteCollection` — returns a compatible iterator

**Key pseudocode:**

```
class FacebookIterator implements ProfileIterator:
    private facebook: Facebook
    private cache: Profile[]
    private currentPosition = 0

    getNext(): Profile:
        if hasMore(): return cache[currentPosition++]

    hasMore(): bool:
        lazyInit()
        return currentPosition < cache.length
```

**Applicability:**

- Collection has a complex internal structure to hide from clients
- Reduce traversal duplication across the app
- Traverse different data structures through a uniform interface

**Pros:** SRP; OCP; parallel iteration (each iterator has its own state); pause and resume iteration.
**Cons:** Overkill for simple collections; may be slower than direct element access.

**Related:** Use with Composite to traverse trees. Factory Method lets collection subclasses return typed iterators. Memento + Iterator: capture iteration state for rollback. Visitor + Iterator: traverse + execute typed operations.

---

## Mediator

*Also known as: Intermediary, Controller*

**Intent:** Reduce chaotic dependencies by routing all communication between components through a mediator object.

**Problem:** Form elements (checkboxes, text fields, buttons) interact with each other directly. Each component is coupled to many others, making them hard to reuse or change independently.

**Solution:** Components don't call each other directly. Instead, they notify the mediator of events. The mediator decides which other components to notify or update in response.

**Structure:**

- `Mediator` interface — `notify(sender, event)`
- `ConcreteMediator` — knows all components; contains the coordination logic
- `Component` — holds a `mediator` reference; calls `mediator.notify()` on events

**Key pseudocode:**

```
class AuthDialog implements Mediator:
    notify(sender, event):
        if sender == loginCheckbox and event == "check":
            if loginCheckbox.checked:
                showLoginForm(); hideRegistrationForm()
            else:
                showRegistrationForm(); hideLoginForm()
        if sender == okButton and event == "click":
            // validate + submit
```

**Applicability:**

- Classes tightly coupled to many others, making changes risky
- Can't reuse a component because it depends on too many others
- Tons of component subclasses created just to reuse behavior in different contexts

**Pros:** SRP; OCP; reduces coupling; easier component reuse.
**Cons:** Mediator can become a god object over time.

**Related:** CoR/Command/Mediator/Observer: different sender-receiver connection strategies. Facade vs Mediator: Facade simplifies access to a subsystem (subsystem unaware of facade); Mediator centralizes component communication (components know only the mediator). Mediator can be implemented using Observer (mediator is publisher, components are subscribers).

---

## Memento

*Also known as: Snapshot*

**Intent:** Save and restore an object's state without exposing its implementation details.

**Problem:** To implement undo, you must snapshot an object's state. But accessing private fields from outside violates encapsulation; making fields public exposes implementation details to all callers.

**Solution:** The originator creates its own snapshot (a `Memento` object with private access). The caretaker stores mementos in a history stack but can only access their metadata — not the internal state. Only the originator can read/restore from a memento.

**Structure:**

- `Originator` — `createSnapshot(): Memento`; `restore(m: Memento)`
- `Memento` — immutable; stores originator's state; only originator can access fields
- `Caretaker` — stores stack of mementos; triggers save/restore at the right time

**Key pseudocode:**

```
class Editor:             // Originator
    createSnapshot(): Snapshot:
        return new Snapshot(this, text, curX, curY, selectionWidth)

class Snapshot:           // Memento
    private editor, text, curX, curY, selectionWidth
    restore():
        editor.setText(text); editor.setCursor(curX, curY)

class Command:            // Caretaker
    makeBackup(): backup = editor.createSnapshot()
    undo(): if backup != null: backup.restore()
```

**Applicability:**

- Produce snapshots to restore previous state (undo, transactions, rollback)
- Direct access to object fields would violate encapsulation

**Pros:** State snapshots without breaking encapsulation; simplifies originator (caretaker manages history).
**Cons:** High RAM if mementos are created frequently; caretakers must manage memento lifecycle.

**Related:** Command + Memento = undo: Command performs, Memento saves state before. Memento + Iterator: snapshot iteration state. Prototype can be a simpler alternative when the state is straightforward.

---

## Observer

*Also known as: Event-Subscriber, Listener*

**Intent:** Define a subscription mechanism to notify multiple objects about state changes.

**Problem:** One object (publisher) needs to inform others (subscribers) of changes without knowing who they are. Polling wastes resources; direct callbacks create tight coupling.

**Solution:** Publisher maintains a subscriber list and exposes `subscribe()`/`unsubscribe()` methods. When state changes, publisher calls `update()` on all subscribers. All subscribers implement a common interface.

**Structure:**

- `Publisher` — maintains `subscribers: Listener[]`; `subscribe()`, `unsubscribe()`, `notify()`
- `Subscriber` interface — `update(data)`
- `ConcreteSubscriber` — reacts to notifications

**Key pseudocode:**

```
class EventManager:
    listeners: map<eventType, Listener[]>
    subscribe(type, listener): listeners[type].add(listener)
    unsubscribe(type, listener): listeners[type].remove(listener)
    notify(type, data):
        foreach listener in listeners[type]: listener.update(data)

class Editor:
    events: EventManager
    saveFile(): file.write(); events.notify("save", file.name)
```

**Applicability:**

- State changes in one object may require updating others; set of dependents unknown or dynamic
- Objects need to notify others for a limited time or in specific circumstances

**Pros:** OCP; establish relations at runtime.
**Cons:** Subscribers notified in random order.

**Related:** CoR/Command/Mediator/Observer: sender-receiver connections. Mediator vs Observer: Mediator centralizes communication; Observer distributes it. Mediator can use Observer internally (mediator = publisher, components = subscribers).

---

## State

**Intent:** Let an object change its behavior when its internal state changes — as if it changed its class.

**Problem:** An object with state-dependent behavior ends up with huge `if/switch` blocks in every method. Adding new states requires changing many methods; the code is fragile and hard to extend.

**Solution:** Create a class for each state. The context object holds a reference to a `State` object and delegates all state-specific calls to it. To transition, replace the state object.

**Structure:**

- `Context` — holds `state: State`; `changeState(s)`; delegates calls to state
- `State` interface — declares all state-specific methods
- `ConcreteState` — implements methods for one state; can trigger transitions via `context.changeState()`

**Key pseudocode:**

```
class AudioPlayer:             // Context
    state: State
    clickPlay(): state.clickPlay()
    changeState(s): this.state = s

abstract class State:
    player: AudioPlayer
    abstract clickPlay(), clickLock(), clickNext()

class ReadyState extends State:
    clickPlay():
        player.startPlayback()
        player.changeState(new PlayingState(player))
```

**Applicability:**

- Object behaves differently depending on current state; states change frequently
- Massive conditionals based on object state fields
- Duplicate code across similar states and transitions

**Pros:** SRP; OCP; eliminate bulky state machine conditionals.
**Cons:** Overkill for simple state machines with few states.

**Related:** State vs Strategy: both use composition to delegate behavior. Strategy makes algorithms independent; State lets states know about and trigger transitions to each other. Bridge/State/Strategy have similar structures but solve different problems.

---

## Strategy

**Intent:** Define a family of interchangeable algorithms, extract each into a class, and make them swappable at runtime.

**Problem:** A class implements multiple variants of an algorithm via large conditionals. Adding a new variant requires modifying the class; different algorithm variants are hard to test or reuse in isolation.

**Solution:** Extract each variant into a `ConcreteStrategy` class implementing a common `Strategy` interface. The `Context` holds a strategy reference and delegates the work to it. The client selects and passes the desired strategy.

**Structure:**

- `Strategy` interface — `execute(data)`
- `ConcreteStrategy` — implements one algorithm variant
- `Context` — holds `strategy: Strategy`; `setStrategy(s)`; calls `strategy.execute()`

**Key pseudocode:**

```
interface Strategy:
    execute(a, b): number

class ConcreteStrategyAdd implements Strategy:
    execute(a, b): return a + b

class Context:
    private strategy: Strategy
    setStrategy(s): this.strategy = s
    executeStrategy(a, b): return strategy.execute(a, b)
```

**Applicability:**

- Use different algorithm variants at runtime
- Many similar classes that differ only in behavior
- Eliminate large conditionals selecting algorithm variants

**Pros:** Swap algorithms at runtime; isolate implementation details; replace inheritance with composition; OCP.
**Cons:** Overkill for a few rarely-changing algorithms; clients must know differences between strategies; modern languages can use lambdas instead.

**Related:** Strategy vs Command: Command converts an operation into an object (supports history, queuing); Strategy swaps algorithms within a context. Strategy vs Template Method: Template Method uses inheritance (static); Strategy uses composition (runtime-swappable). State is an extension of Strategy: State lets states trigger transitions; Strategy keeps algorithms independent.

---

## Template Method

**Intent:** Define the skeleton of an algorithm in a base class; let subclasses override specific steps without changing the overall structure.

**Problem:** Multiple classes implement the same algorithm but differ in details. Code is duplicated across classes; changes to the shared structure require modifying every class.

**Solution:** Define the invariant algorithm structure in a `templateMethod()` in the base class. Break it into steps; declare abstract steps for subclasses to implement. Optional "hook" methods have empty default implementations for subclasses to optionally override.

**Structure:**

- `AbstractClass` — `templateMethod()` (final); abstract step methods; optional hook methods with default implementations
- `ConcreteClass` — implements abstract steps; may override hooks

**Key pseudocode:**

```
class GameAI:
    method turn():           // Template method
        collectResources()
        buildStructures()    // abstract
        buildUnits()         // abstract
        attack()

    method collectResources():
        foreach s in builtStructures: s.collect()  // default

    abstract buildStructures()
    abstract buildUnits()

class OrcsAI extends GameAI:
    buildStructures(): // Build farms → barracks → stronghold
    buildUnits(): // Build peons and grunts
```

**Applicability:**

- Let clients extend parts of an algorithm but not its overall structure
- Several classes share the same algorithm with minor differences

**Pros:** Clients override only specific parts; pull shared code into superclass.
**Cons:** Skeleton constrains clients; risks violating Liskov Substitution; harder to maintain with many steps.

**Related:** Factory Method is a specialization of Template Method. Template Method uses *inheritance* (static); Strategy uses *composition* (runtime-swappable).

---

## Visitor

**Intent:** Separate an algorithm from the objects it operates on; add new operations without modifying the classes.

**Problem:** You want to add a new operation (e.g., XML export) to a class hierarchy without modifying the classes. You can't modify them (third-party, production code), or doing so would bloat them with unrelated behavior.

**Solution:** Place the new behavior in a separate `Visitor` class. Each element class gets an `accept(visitor)` method that calls the appropriate visitor method (double dispatch). To add a new operation, create a new visitor class — no element changes needed.

**Double dispatch:** Instead of checking the element's type in the visitor, the element calls `visitor.visitMyType(this)`, routing to the correct method automatically.

**Structure:**

- `Visitor` interface — one `visit(ElementType)` method per concrete element class
- `ConcreteVisitor` — implements the operation for each element type
- `Element` interface — `accept(v: Visitor)`
- `ConcreteElement` — `accept(v): v.visitConcreteElement(this)`

**Key pseudocode:**

```
interface Visitor:
    visitDot(d: Dot)
    visitCircle(c: Circle)
    visitRectangle(r: Rectangle)

class XMLExportVisitor implements Visitor:
    visitDot(d): // export dot's ID and coordinates
    visitCircle(c): // export circle's ID, center, radius

class Circle implements Shape:
    accept(v: Visitor): v.visitCircle(this)

// Client:
foreach shape in allShapes:
    shape.accept(exportVisitor)
```

**Applicability:**

- Perform operations on all elements of a complex object structure
- Extract auxiliary behaviors to keep element classes focused
- A behavior makes sense only for some subclasses; implement it selectively in the visitor

**Pros:** OCP (add visitors without changing elements); SRP; accumulate state across the traversal.
**Cons:** Must update all visitors when the element hierarchy changes; visitors may lack access to private members.

**Related:** Visitor is a powerful version of Command (operates on multiple classes). Use with Composite to operate on entire trees. Use with Iterator to traverse + execute typed operations.
