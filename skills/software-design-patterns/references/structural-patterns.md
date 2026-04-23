# Structural Patterns — Full Reference

---

## Adapter

*Also known as: Wrapper*

**Intent:** Allow objects with incompatible interfaces to collaborate.

**Problem:** You want to use a third-party class but its interface doesn't match the one expected by your code. You can't change the third-party source.

**Solution:** Create an adapter class that implements the target interface and wraps the incompatible service object. It translates calls from the target interface into calls the service understands.

**Two variants:**

- **Object adapter** — holds a reference to the service (composition); works in all languages
- **Class adapter** — inherits from both (multiple inheritance); C++ only

**Structure:**

- `Client` — calls methods on the target interface
- `Target` interface — what the client expects
- `Service` — incompatible class being wrapped
- `Adapter` — implements `Target`, holds a `Service` reference, translates calls

**Key pseudocode:**

```
class SquarePegAdapter extends RoundPeg:
    private field peg: SquarePeg
    constructor(peg): this.peg = peg
    getRadius(): return peg.getWidth() * sqrt(2) / 2
```

**Applicability:**

- Use an existing class with an incompatible interface
- Reuse subclasses that lack common functionality without modifying the superclass

**Pros:** SRP; OCP (add new adapters without touching existing code).
**Cons:** Added complexity; sometimes simpler to just change the service.

**Related:** Bridge is designed up-front; Adapter fixes incompatibilities in existing code. Adapter gives a *different* interface; Decorator gives an *enhanced* interface; Proxy keeps the *same* interface. Facade wraps a whole subsystem; Adapter usually wraps one object.

---

## Bridge

**Intent:** Split a large class or closely related classes into two separate hierarchies — abstraction and implementation — that can evolve independently.

**Problem:** Extending a class in two orthogonal dimensions (e.g. Shape × Color) causes a combinatorial explosion of subclasses.

**Solution:** Extract one dimension into a separate hierarchy. The original class holds a *reference* (bridge) to an object from the extracted hierarchy rather than inheriting from it.

**Structure:**

- `Abstraction` — high-level control; holds a reference to an `Implementation`
- `RefinedAbstraction` — extends abstraction with extra behaviors
- `Implementation` interface — low-level primitive operations
- `ConcreteImplementation` — platform-specific code

**Key pseudocode:**

```
class RemoteControl:          // Abstraction
    protected device: Device  // bridge field
    togglePower(): device.enable() / device.disable()

class AdvancedRemote extends RemoteControl:
    mute(): device.setVolume(0)

interface Device:
    enable(); disable(); getVolume(); setVolume()

class Tv implements Device: ...
class Radio implements Device: ...
```

**Applicability:**

- Avoid class explosion from two independent extension dimensions
- Switch implementations at runtime
- Divide a monolithic class that varies in multiple directions

**Pros:** Platform-independent; OCP; SRP; can swap implementations at runtime.
**Cons:** Can complicate a cohesive class.

**Related:** Adapter fixes incompatibilities after the fact; Bridge is planned up-front. Can combine with Abstract Factory to manage compatible implementations. Builder can act as Bridge (director = abstraction, builders = implementations).

---

## Composite

*Also known as: Object Tree*

**Intent:** Compose objects into tree structures and treat individual objects and composites uniformly.

**Problem:** You have part-whole hierarchies (files and folders, UI widgets and containers). Client code needs to handle both leaf nodes and containers without special-casing.

**Solution:** Define a common `Component` interface for both leaves and containers. Containers (`Composite`) hold a list of children — also `Component` — and delegate operations to them recursively.

**Structure:**

- `Component` interface — common operations (`draw()`, `getPrice()`, etc.)
- `Leaf` — no children; does actual work
- `Composite` — holds `children: Component[]`; delegates to children + aggregates results

**Key pseudocode:**

```
interface Graphic:
    draw(); move(x, y)

class Dot implements Graphic:
    draw(): // draws a dot

class CompoundGraphic implements Graphic:
    children: Graphic[]
    draw(): foreach child: child.draw()
    add(child): children.add(child)
```

**Applicability:**

- Represent part-whole hierarchies
- Client code should treat simple and complex elements uniformly

**Pros:** Polymorphism + recursion make tree operations clean; OCP.
**Cons:** Hard to restrict component interface when component types differ too much.

**Related:** Builder can construct Composite trees. Chain of Responsibility often traverses Composite trees. Iterator traverses them. Visitor executes operations on the whole tree. Flyweight can be used for shared leaf nodes. Decorator extends a node's behavior without changing its type.

---

## Decorator

*Also known as: Wrapper*

**Intent:** Attach new behaviors to objects at runtime by wrapping them in decorator objects.

**Problem:** You need optional, stackable features (e.g. notifications via Email + SMS + Slack). Using inheritance causes a subclass explosion; features are static and can't be combined.

**Solution:** Create a `BaseDecorator` that implements the same interface as the wrapped component and holds a reference to it. Concrete decorators extend the base decorator, executing extra behavior before/after delegating to the wrapped object. Decorators can be stacked in any order.

**Structure:**

- `Component` interface — operations common to wrappers and wrapped objects
- `ConcreteComponent` — base implementation
- `BaseDecorator` — holds `wrappee: Component`; delegates all calls
- `ConcreteDecorator` — overrides methods to add behavior before/after delegation

**Key pseudocode:**

```
class DataSourceDecorator implements DataSource:
    protected wrappee: DataSource
    writeData(d): wrappee.writeData(d)   // delegate

class EncryptionDecorator extends DataSourceDecorator:
    writeData(d):
        encrypted = encrypt(d)
        wrappee.writeData(encrypted)

// Stacking:
source = new EncryptionDecorator(new CompressionDecorator(new FileDataSource("file")))
```

**Applicability:**

- Add behaviors to objects without subclassing
- Add/remove responsibilities at runtime
- When inheritance is unavailable (final class)

**Pros:** Add behavior without subclassing; add/remove at runtime; combine decorators freely; SRP.
**Cons:** Hard to remove a specific wrapper from the stack; order-dependent; verbose initial configuration.

**Related:** Adapter provides a different interface; Decorator extends the same interface. Proxy controls access (same interface); Decorator enriches behavior. CoR and Decorator both use recursive composition — but CoR can stop the chain; Decorator cannot. Composite and Decorator both use recursive composition — but Composite aggregates; Decorator wraps one child and adds behavior.

---

## Facade

**Intent:** Provide a simplified interface to a complex subsystem.

**Problem:** Client code must initialize many subsystem objects, call them in the right order, and manage dependencies — leading to tight coupling to subsystem internals.

**Solution:** Create a `Facade` class that exposes only the high-level operations clients actually need. The facade delegates to the appropriate subsystem objects internally. Subsystem classes remain unchanged and unaware of the facade.

**Structure:**

- `Facade` — simplified interface; knows which subsystem parts to use and in what order
- `Subsystem classes` — complex internals; communicate with each other directly
- `Client` — calls only the facade

**Key pseudocode:**

```
class VideoConverter:
    convert(filename, format): File:
        file = new VideoFile(filename)
        codec = CodecFactory.extract(file)
        buffer = BitrateReader.read(filename, codec)
        result = BitrateReader.convert(buffer, destCodec)
        return AudioMixer.fix(result)

// Client only calls:
mp4 = new VideoConverter().convert("cats.ogg", "mp4")
```

**Applicability:**

- Provide a simple interface to a complex subsystem
- Layer a subsystem and define entry points per layer
- Reduce coupling between subsystems

**Pros:** Isolates clients from subsystem complexity.
**Cons:** Can become a god object coupled to all subsystem classes.

**Related:** Facade defines a *new* interface; Adapter makes an existing interface usable. Facade can become a Singleton. Facade vs Mediator: Facade simplifies access to a subsystem (subsystem unaware of facade); Mediator centralizes communication between components (components only know the mediator). Proxy and Facade both buffer a complex entity — but Proxy keeps the same interface as the service.

---

## Flyweight

*Also known as: Cache*

**Intent:** Share common state between many fine-grained objects to reduce memory usage.

**Problem:** Huge numbers of similar objects (bullets, particles, trees) each store duplicate data (texture, color, sprite), consuming excessive RAM.

**Solution:** Split object state into **intrinsic** (shared, immutable — stored in the flyweight) and **extrinsic** (unique per context — stored/passed by the client). Multiple context objects reference a single flyweight. A `FlyweightFactory` manages the pool of flyweights.

**Structure:**

- `Flyweight` — stores intrinsic state; methods receive extrinsic state as parameters
- `FlyweightFactory` — pool manager; returns existing flyweight or creates new one
- `Context` — stores extrinsic state + reference to a flyweight

**Key pseudocode:**

```
class TreeType:             // Flyweight
    field name, color, texture
    draw(canvas, x, y): ...

class TreeFactory:
    static treeTypes: map
    static getTreeType(name, color, texture):
        if not in map: create and add
        return from map

class Tree:                 // Context
    field x, y
    field type: TreeType
    draw(canvas): type.draw(canvas, x, y)
```

**Applicability:**

- Program has a huge number of similar objects draining RAM
- Objects contain duplicate state that can be extracted and shared

**Pros:** Potentially huge RAM savings.
**Cons:** CPU may increase if extrinsic state must be recalculated; code becomes more complex.

**Related:** Composite tree shared leaves can be Flyweights. Flyweight shows how to make many small objects; Facade shows how to make one object represent a subsystem. Flyweight ≠ Singleton: Singleton has one instance; Flyweight can have multiple with different intrinsic states; Flyweight is immutable.

---

## Proxy

**Intent:** Provide a substitute that controls access to the original object.

**Problem:** You need to add lazy initialization, access control, logging, caching, or remote delegation around an object, but can't or shouldn't modify that object directly.

**Solution:** Create a `Proxy` class with the same interface as the real service. The proxy intercepts calls, performs pre/post processing, and delegates to the real service when appropriate.

**Common proxy types:**

- **Virtual proxy** — lazy initialization of a heavyweight object
- **Protection proxy** — access control (check credentials before delegating)
- **Remote proxy** — handles network communication to a remote service
- **Logging proxy** — logs requests before forwarding
- **Caching proxy** — caches results; returns cached value on repeated calls
- **Smart reference** — tracks references and frees the service when unused

**Structure:**

- `ServiceInterface` — implemented by both `Service` and `Proxy`
- `Service` — real business logic
- `Proxy` — holds `service: Service`; pre/post processing; optional lazy creation

**Key pseudocode:**

```
class CachedYouTubeClass implements ThirdPartyYouTubeLib:
    private service: ThirdPartyYouTubeLib
    private listCache, videoCache

    listVideos():
        if listCache == null: listCache = service.listVideos()
        return listCache
```

**Applicability:**

- Lazy init of heavy objects
- Access control
- Remote service calls
- Logging / caching / lifecycle management

**Pros:** Control service without clients knowing; lifecycle management; works even if service is unavailable; OCP.
**Cons:** Added classes; potential response delay.

**Related:** Adapter gives a different interface; Proxy keeps the same interface; Decorator enhances the interface. Facade buffers a subsystem; Proxy buffers a single service with the same interface. Decorator composition is always client-controlled; Proxy usually manages the service lifecycle itself.
