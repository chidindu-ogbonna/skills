# Creational Patterns — Full Reference

---

## Factory Method

*Also known as: Virtual Constructor*

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Problem:** Code is coupled to a concrete class (`new Truck()`). Adding new product types forces changes everywhere that construction happens.

**Solution:** Replace `new ConcreteProduct()` calls with a factory method. Subclasses override the factory method to return their own product type. All products share a common interface.

**Structure:**

- `Creator` — declares abstract `factoryMethod(): Product`; contains business logic that calls the factory method
- `ConcreteCreator` — overrides factory method, returns a `ConcreteProduct`
- `Product` — common interface for all products
- `ConcreteProduct` — implements `Product`

**Key pseudocode:**

```
class Dialog:
    abstract method createButton(): Button   // factory method
    method render():
        btn = createButton()                 // uses the factory method
        btn.render()

class WindowsDialog extends Dialog:
    method createButton(): Button:
        return new WindowsButton()
```

**Applicability:**

- Don't know the exact class to instantiate at compile time
- Want to let library users extend internal components
- Want to reuse existing objects (object pool)

**Pros:** Decouples creator from products; OCP — add new products without touching existing code.
**Cons:** Requires creating many subclasses.

**Related:** Evolves into Abstract Factory / Builder / Prototype. Specialization of Template Method.

---

## Abstract Factory

**Intent:** Produce families of related objects without specifying their concrete classes.

**Problem:** Code must work with multiple product families (e.g. Windows UI vs Mac UI). Mixing families produces inconsistent results.

**Solution:** Declare an abstract factory interface with a creation method per product type. Each concrete factory implements the interface for one family. Client code only uses the abstract interface.

**Structure:**

- `AbstractFactory` — `createButton()`, `createCheckbox()`
- `ConcreteFactory` (e.g. `WinFactory`, `MacFactory`) — creates the family's products
- `AbstractProduct` (`Button`, `Checkbox`) — product interfaces
- `ConcreteProduct` — `WinButton`, `MacButton`, etc.

**Key pseudocode:**

```
interface GUIFactory:
    createButton(): Button
    createCheckbox(): Checkbox

class WinFactory implements GUIFactory:
    createButton(): return new WinButton()
    createCheckbox(): return new WinCheckbox()
```

**Applicability:**

- Code must work with various product families
- Enforce that products of the same family are used together

**Pros:** Products are compatible; OCP; SRP.
**Cons:** Adding a new product type requires changing every factory.

**Related:** Often based on Factory Methods; alternative to Facade for hiding subsystem creation; can be combined with Bridge; implementations can be Singletons.

---

## Builder

**Intent:** Construct complex objects step by step; same construction code can produce different representations.

**Problem:** A class with many optional fields leads to a telescoping constructor or a bloated object with many null fields.

**Solution:** Extract construction into a `Builder` object with step methods (`setEngine()`, `setGPS()`). An optional `Director` class defines the construction sequence. Client calls steps in the needed order and retrieves the product.

**Structure:**

- `Builder` interface — step methods + `reset()`
- `ConcreteBuilder` — implements steps, holds the product, exposes `getProduct()`
- `Director` — knows how to call steps in the right order
- `Product` — the complex object being built

**Key pseudocode:**

```
class CarBuilder implements Builder:
    method reset(): this.car = new Car()
    method setSeats(n): ...
    method setEngine(e): ...
    method getProduct(): Car:
        p = this.car; this.reset(); return p

class Director:
    method constructSportsCar(b: Builder):
        b.reset(); b.setSeats(2); b.setEngine(SportEngine())
```

**Applicability:**

- Eliminate telescoping constructors
- Need different representations using the same construction process
- Building Composite trees step by step

**Pros:** Step-by-step construction; reusable construction code; SRP.
**Cons:** Increased complexity (multiple new classes).

**Related:** Can build Composite trees; combine with Bridge (director = abstraction, builders = implementations); implementations can be Singletons.

---

## Prototype

*Also known as: Clone*

**Intent:** Copy existing objects without depending on their concrete class.

**Problem:** You want to duplicate an object but can't access its private fields from outside, or its concrete class is unknown (known only by interface).

**Solution:** Declare a `clone()` method on the object. The object copies itself — including private fields — and returns the clone. An optional `PrototypeRegistry` stores pre-configured prototypes for lookup.

**Structure:**

- `Prototype` interface — `clone(): Prototype`
- `ConcretePrototype` — implements `clone()` by calling a copy constructor
- `PrototypeRegistry` (optional) — `name → prototype` map

**Key pseudocode:**

```
abstract class Shape:
    constructor Shape(source: Shape):
        this.X = source.X; this.Y = source.Y
    abstract clone(): Shape

class Circle extends Shape:
    constructor Circle(source: Circle):
        super(source); this.radius = source.radius
    clone(): return new Circle(this)
```

**Applicability:**

- Code shouldn't depend on concrete classes of objects to copy
- Reduce subclass explosion by cloning pre-configured prototypes

**Pros:** Clone without coupling; eliminate repeated initialization; alternative to inheritance for configuration presets.
**Cons:** Circular references make cloning tricky.

**Related:** Can save copies of Commands into history. Composite/Decorator-heavy designs benefit from Prototype for cloning structures. Simpler alternative to Memento for straightforward objects.

---

## Singleton

**Intent:** Ensure a class has only one instance and provide a global access point to it.

**Problem:** Multiple instantiations of a shared resource (DB connection, logger) cause inconsistency. Global variables are unsafe (overwritable).

**Solution:** Make the constructor private. Provide a static `getInstance()` method that creates the instance on first call and caches it. All subsequent calls return the cached instance.

**Structure:**

- Private static field `instance`
- Private constructor
- Public static `getInstance()` with lazy init (+ thread lock for thread safety)

**Key pseudocode:**

```
class Database:
    private static instance: Database
    private constructor Database(): // connect...

    static getInstance(): Database:
        if instance == null:
            lock(); if instance == null: instance = new Database()
        return instance
```

**Applicability:**

- Single shared instance must exist (DB, config, logger)
- Stricter control than global variables

**Pros:** Single instance guaranteed; controlled global access; lazy initialization.
**Cons:** Violates SRP; hard to unit test (can't mock); needs special handling in multithreaded environments.

**Related:** Facade can be turned into a Singleton. Flyweight differs: multiple instances with different intrinsic states; Flyweight is immutable. Abstract Factory / Builder / Prototype can be implemented as Singletons.
