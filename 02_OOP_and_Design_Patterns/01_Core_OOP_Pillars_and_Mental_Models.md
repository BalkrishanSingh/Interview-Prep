# Module 02: OOP — Core Pillars & Object-Oriented Mental Models

---

## 1. What is Object-Oriented Programming?

Object-Oriented Programming (OOP) is a software design paradigm that models systems around **objects**—autonomous computational entities that bundle **internal state** (data attributes) and **behavior** (operations and methods) together behind protected boundaries.

### Procedural vs. Object-Oriented Thinking

```
PROCEDURAL MODEL (Separated Data & Logic):
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Raw Data    │ ───► │ Procedure A  │ ───► │ Procedure B  │
│  Structures  │      │ (Mutates)    │      │ (Mutates)    │
└──────────────┘      └──────────────┘      └──────────────┘
• State is globally accessible and mutable by any function.
• Changes to data structures ripple through and break unrelated functions.

OBJECT-ORIENTED MODEL (Encapsulated State & Behavior):
┌──────────────────────────────────────────────────────────┐
│ OBJECT                                                   │
│   Private State:   [balance: $500, status: ACTIVE]       │
│   Public Behavior: deposit(), withdraw(), getBalance()   │
└──────────────────────────────────────────────────────────┘
• State is protected inside the object boundary.
• Outside code interacts only by invoking behavior through public interfaces.
```

---

## 2. The Four Foundational Pillars of OOP

```
                   ┌───────────────────────────────┐
                   │    THE 4 PILLARS OF OOP       │
                   └───────────────────────────────┘
                     │          │          │          │
        ┌────────────┘          │          │          └────────────┐
        ▼                       ▼          ▼                       ▼
 ┌──────────────┐        ┌─────────────┐ ┌─────────────┐    ┌──────────────┐
 │ Encapsulation│        │ Abstraction │ │ Inheritance │    │ Polymorphism │
 │  Data Hiding │        │  Essential  │ │  Code Reuse │    │ Many Forms   │
 │  & Invariants│        │  Interfaces │ │  & "Is-A"   │    │  & Overrides │
 └──────────────┘        └─────────────┘ └─────────────┘    └──────────────┘
```

---

### 2.1 Encapsulation: Data Hiding & Invariant Protection

**Encapsulation** bundles data attributes and the operations that mutate that data into a cohesive unit, while strictly controlling external access.

#### Core Objectives:
1. **Invariant Preservation**: An invariant is a business condition that must always remain true for an object to be in a valid state (e.g., `account.balance >= 0` or `cart.total_items >= 0`). Encapsulation ensures external code cannot bypass validation and force the object into a corrupt state.
2. **Implementation Hiding**: Internal data structures (e.g., whether a queue is implemented using a linked list or dynamic array) can be refactored without breaking any external callers.

#### Universal Access Control Specifiers:
| Access Modifier | Accessibility Scope | Purpose |
| :--- | :--- | :--- |
| **`public`** | Accessible anywhere by any caller. | Public API contract of the class. |
| **`protected`** | Accessible only within the class and its derived subclasses. | Extension points for specialized descendants. |
| **`private`** | Accessible strictly within the declaring class itself. | Internal state and implementation helper methods. |
| **`internal` / Package** | Accessible within the same package or module. | Cohesive subsystem collaboration. |

#### True Encapsulation vs. The "Anemic" Getter/Setter Anti-Pattern:
Simply declaring all fields `private` and generating public `get_field()` and `set_field()` methods for each is **not** true encapsulation. It exposes internal representation and allows external code to perform calculations on the object's behalf. True encapsulation exposes **intent and behavior**, not raw data.

```python
# Clean Encapsulation: Object protects its own invariants
class BankAccount:
    def __init__(self, account_holder: str, initial_deposit: float):
        if initial_deposit < 0:
            raise ValueError("Initial deposit cannot be negative.")
        self._holder = account_holder
        self._balance = initial_deposit

    def get_balance(self) -> float:
        return self._balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit amount must be strictly positive.")
        self._balance += amount

    def withdraw(self, amount: float) -> bool:
        if amount <= 0:
            raise ValueError("Withdrawal amount must be strictly positive.")
        if amount > self._balance:
            return False  # Invariant protected: cannot overdraw
        self._balance -= amount
        return True
```

---

### 2.2 Abstraction: Essential Contracts vs. Internal Mechanics

**Abstraction** exposes only the essential characteristics and interfaces of an entity to the outside world, suppressing implementation details.

- **The Contract**: A caller needs to know *what* an operation achieves, not *how* it achieves it under the hood.
- **Mental Model**: A driver turns the steering wheel to navigate. The steering wheel abstracts away the rack-and-pinion gear, hydraulic pumps, power steering motors, and tire friction physics.
- **Abstract Data Types (ADTs)**: A `Stack` ADT guarantees `push(x)` and `pop()` operations in LIFO order. Whether it is backed by an array, a doubly linked list, or memory mapped files is an abstracted detail.

```python
from abc import ABC, abstractmethod

# Abstract Interface: Declares the contract
class PaymentGateway(ABC):
    @abstractmethod
    def process_payment(self, amount_in_cents: int) -> bool:
        """Processes transaction. Must be implemented by concrete gateways."""
        pass

# Concrete Implementation A: External API via HTTPS
class StripeGateway(PaymentGateway):
    def process_payment(self, amount_in_cents: int) -> bool:
        # Handles TLS handshake, JSON payload serialization, HMAC signing
        print(f"Submitting {amount_in_cents} cents to Stripe API.")
        return True

# Concrete Implementation B: Direct Banking Protocol
class WireTransferGateway(PaymentGateway):
    def process_payment(self, amount_in_cents: int) -> bool:
        # Formats SWIFT / ISO-20022 banking message
        print(f"Dispatching wire transfer for {amount_in_cents} cents.")
        return True
```

---

### 2.3 Inheritance & Subtyping

**Inheritance** allows a new class (derived/subclass) to inherit attributes and behaviors from an existing class (base/superclass), establishing an **"Is-A"** taxonomy.

- **Code Reuse**: Eliminates duplicate logic by hoisting shared methods into a common ancestor.
- **Subtyping vs. Implementation Inheritance**:
  - *Implementation Inheritance*: Inheriting executable code from a parent class.
  - *Subtyping (Interface Inheritance)*: Guaranteeing that an instance of the subclass can be substituted wherever the superclass is expected without breaking system behavior (**Liskov Substitution Principle**).
- **The Fragile Base Class Problem**: A major hazard of deep inheritance hierarchies. Modifying a method in the base class can inadvertently break assumptions and invariants in descendant subclasses five levels down.

---

### 2.4 Polymorphism ("Many Forms")

**Polymorphism** allows code to treat objects of different concrete types through a uniform interface, with each object executing behavior appropriate to its specific type.

The two fundamental forms of polymorphism:

| Form | Mechanism | Resolution Time | Key Examples |
| :--- | :--- | :--- | :--- |
| **Compile-Time (Static)** | **Method Overloading** | Compile-Time (Early Binding) | Multiple methods sharing the same name but differing in parameter count or parameter types. The compiler matches the exact call signature at build time. |
| **Runtime (Dynamic)** | **Method Overriding** | Runtime (Late Binding) | A subclass provides its own specific implementation of a method already declared in a superclass or interface. The runtime resolves which method to invoke based on the actual concrete object. |

```python
# Universal Polymorphism: Client code operates against the common base
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius * self.radius

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

# Client operates polymorphically without knowing concrete shape types
def calculate_total_surface(shapes: list[Shape]) -> float:
    return sum(shape.area() for shape in shapes)
```

---

## 3. How Dynamic Dispatch Works: Virtual Method Tables (VTables)

How does a program know which overridden method to execute when calling `shape.area()` if the variable `shape` is statically typed as the generic base class?

### 3.1 Static Dispatch vs. Dynamic Dispatch

- **Static Dispatch (Early Binding)**: Used for non-virtual methods, private methods, and static methods. The compiler determines the exact memory address of the target function at compile time. At machine level, this generates a direct `CALL 0x00401020` instruction with zero runtime lookup cost.
- **Dynamic Dispatch (Late Binding)**: Used for virtual / overridable methods. The actual method to execute depends on the runtime concrete type of the object. This resolution is achieved via a **Virtual Method Table (vtable)**.

### 3.2 The VTable Mechanism

1. **Class-Level Table (`vtable`)**:
   For every class that declares or overrides virtual methods, the compiler creates a static array of function pointers called the `vtable`.
2. **Object-Level Pointer (`vptr`)**:
   Every instance of that class contains an invisible pointer (the `vptr`) in its object memory header that points directly to its class's `vtable`.

```
                    OBJECT INSTANCES IN HEAP                CLASS VTABLES (STATIC MEMORY)
                  ┌───────────────────────────┐           ┌─────────────────────────────┐
circle_instance   │ Header (vptr)             │ ────────► │ Circle vtable               │
                  ├───────────────────────────┤           │ [0] &Circle::area           │
                  │ radius: 5.0               │           └─────────────────────────────┘
                  └───────────────────────────┘
                  ┌───────────────────────────┐           ┌─────────────────────────────┐
rect_instance     │ Header (vptr)             │ ────────► │ Rectangle vtable            │
                  ├───────────────────────────┤           │ [0] &Rectangle::area        │
                  │ width: 4.0, height: 6.0   │           └─────────────────────────────┘
                  └───────────────────────────┘
```

### 3.3 What Happens When `shape.area()` Executes?
1. The program inspects the memory address of the target object `shape`.
2. It reads the hidden `vptr` located in the object's header.
3. It indexes into the resolved `vtable` at the predetermined offset for `area()` (e.g., slot `0`).
4. It dereferences the function pointer stored at that slot and jumps to the concrete code.

**Performance Trade-off**: Dynamic dispatch costs an extra memory dereference and can prevent compiler function inlining. In modern software, this sub-nanosecond indirection is universally embraced for the massive architectural decoupling it enables.

---

## 4. The Diamond Problem & How Languages Resolve It

The **Diamond Problem** is the classic dilemma that arises in multiple inheritance when a class inherits from two parent classes that both derive from a single common ancestor:

```
                    ┌──────────────────┐
                    │      Class A     │
                    │   def ping():    │
                    │     return "A"   │
                    └──────────────────┘
                             ▲
                    ┌────────┴────────┐
                    │                 │
             ┌─────────────┐   ┌─────────────┐
             │   Class B   │   │   Class C   │
             │ def ping(): │   │ def ping(): │
             │  return "B" │   │  return "C" │
             └─────────────┘   └─────────────┘
                    ▲                 ▲
                    └────────┬────────┘
                             │
                    ┌──────────────────┐
                    │     Class D      │
                    │   d.ping() = ?   │
                    └──────────────────┘
```

### 4.1 The Two Fatal Ambiguities
1. **Method Ambiguity**: If both `B` and `C` override `ping()`, which version should `d.ping()` invoke?
2. **State Duplication**: If `A` defines a member variable `counter`, does an instance of `D` allocate memory for **two** distinct copies of `counter` (one via `B` and one via `C`), or **one**?

---

### 4.2 Language Resolution Models

#### Model 1: The Interface Solution (Java, C#, Go)
- **Rule**: Multiple class inheritance of **state** is completely prohibited. A class can extend only **one** parent class.
- **Extensibility**: A class can implement **multiple interfaces** (contracts without instance state). Because interfaces contain no instance variables, state duplication is impossible. If two interfaces declare identical default methods, the implementing subclass is forced to explicitly override and resolve the collision.

#### Model 2: Virtual Inheritance (C++)
- **Rule**: Classes specify `class B : virtual public A` and `class C : virtual public A`.
- **Mechanics**: The compiler alters the memory layout so that only a single shared sub-object of `A` is allocated for `D`. A virtual base pointer indexes the shared base, eliminating duplicate state.

#### Model 3: Method Resolution Order (MRO) via C3 Linearization (Python, Dylan)
Languages supporting multiple implementation inheritance use a deterministic algorithm called **C3 Linearization** to produce a flat, unambiguous search list of classes for any hierarchy.

The algorithm guarantees three essential properties:
1. **Subclasses before Parents**: Children are always searched before their ancestors.
2. **Local Precedence**: If a class is declared as `class D(B, C)`, the search order strictly respects `B` before `C`.
3. **Monotonicity**: If class $X$ precedes class $Y$ in the resolution order of a parent class, $X$ will always precede $Y$ in the resolution order of all descendant classes.

```python
class A:
    def ping(self): return "A"

class B(A):
    def ping(self): return "B"

class C(A):
    def ping(self): return "C"

class D(B, C):
    pass

# Deterministic resolution order: D -> B -> C -> A -> object
print([cls.__name__ for cls in D.__mro__])
# Output: ['D', 'B', 'C', 'A', 'object']
# Therefore, d.ping() resolves unambiguously to B's implementation.
```

---

## 5. Static vs. Instance Semantics

Understanding where state and logic live in memory is fundamental to robust Object-Oriented design.

```
┌────────────────────────────────────────────────────────┐
│ CLASS METADATA (Static Memory / Metaspace)             │
│ • Static Fields (e.g. BankAccount.interest_rate)       │
│ • Static Methods (e.g. BankAccount.calculate_tax)      │
│ • Class VTable references                              │
│ • Exactly ONE copy per class shared by all instances   │
└────────────────────────────────────────────────────────┘
                           ▲
                           │ referenced by
┌──────────────────────────┴─────────────────────────────┐
│ HEAP MEMORY (Instance Space)                           │
│ • Instance 1: [balance: $100, owner: "Alice"]          │
│ • Instance 2: [balance: $500, owner: "Bob"]            │
│ • Each object holds its own distinct member variables  │
└────────────────────────────────────────────────────────┘
```

### 5.1 Key Distinctions

| Dimension | Instance Members | Static (Class) Members |
| :--- | :--- | :--- |
| **Binding Time** | Dynamic (Late Binding via instance reference). | Static (Early Binding via class reference at compile time). |
| **Object Receiver** | Requires an instantiated object (`this` / `self`). | Belongs to the class type; callable without instantiating an object. |
| **State Access** | Can read and mutate both instance state and static state. | Can only access static state; cannot access instance state. |
| **Polymorphism** | Fully overridable through dynamic dispatch. | **Cannot be polymorphically overridden**. Redefining a static method in a subclass causes **Method Hiding**, not overriding. |

### 5.2 Why Static Methods Cannot Be Overridden
Static methods are bound to the class type declared at compile time, not the concrete object in memory. If a subclass declares a static method with the same signature as its superclass, callers referencing the base class type will still invoke the base class version.

### 5.3 Legitimate Uses of Static Members
1. **Pure Utility Functions**: Stateless calculations that depend solely on their inputs (e.g., `Math.max(a, b)`).
2. **Static Factory Methods**: Providing descriptive, controlled constructors:
   ```python
   class User:
       def __init__(self, username: str, role: str):
           self.username = username
           self.role = role

       @classmethod
       def create_admin(cls, username: str):
           """Static factory method providing explicit intent."""
           return cls(username, role="ADMIN")

       @classmethod
       def create_guest(cls):
           """Static factory method with default configuration."""
           return cls("guest_user", role="READ_ONLY")
   ```

---

## 6. Core Object-Oriented Design Heuristics

Design patterns and architectural best practices stem from a small set of foundational heuristics.

---

### 6.1 Favor Object Composition Over Class Inheritance

> *"Inheritance is compile-time and rigid; Composition is runtime and flexible."*

- **Inheritance ("Is-A")**: Tightly couples the subclass to the internal structure of the superclass. A change in the superclass can inadvertently break subclass invariants (the Fragile Base Class problem).
- **Composition ("Has-A")**: Objects reference other objects and delegate tasks through abstract interfaces. Behavior can be swapped dynamically at runtime without altering class hierarchies.

```python
# POOR: Rigid inheritance hierarchy
class Engine:
    def ignite(self): pass

class Car(Engine):  # Anti-pattern: A Car IS NOT an Engine!
    pass

# CLEAN: Composition with behavioral delegation
class CombustionEngine:
    def start(self): return "V8 combustion engine running."

class ElectricMotor:
    def start(self): return "Electric motor humming silently."

class Vehicle:
    def __init__(self, motor):
        self._motor = motor  # Composition: Vehicle HAS-A motor

    def drive(self):
        print(self._motor.start())

# Behaviors are interchangeable at runtime without modifying Vehicle
car = Vehicle(ElectricMotor())
car.drive()
```

---

### 6.2 High Cohesion & Loose Coupling

The twin pillars of clean software architecture:

```
LOW COHESION & TIGHT COUPLING (Fragile):
┌────────────────────────────────────────────────────────┐
│ Monolithic OrderProcessor                              │
│ • Validates inventory    • Queries SQL database        │
│ • Charges credit card    • Generates PDF invoices      │
│ • Dispatches emails      • Logs raw metrics            │
└────────────────────────────────────────────────────────┘
Any change to database schema or email provider forces modifications to OrderProcessor!

HIGH COHESION & LOOSE COUPLING (Robust):
┌──────────────────┐    ┌────────────────────┐    ┌────────────────────┐
│ PaymentService   │    │ InventoryManager   │    │ NotificationSender │
│ (Focus: Billing) │    │ (Focus: Stock)     │    │ (Focus: Delivery)  │
└──────────────────┘    └────────────────────┘    └────────────────────┘
         ▲                        ▲                         ▲
         └────────────────────────┼─────────────────────────┘
                   ┌──────────────────────────────┐
                   │ OrderCoordinator (Mediator)  │
                   └──────────────────────────────┘
```

- **Cohesion**: How focused and single-purpose the responsibilities of a single class are. High cohesion means all attributes and methods are tightly aligned toward fulfilling one distinct responsibility.
- **Coupling**: The degree of direct dependency between different classes. Loose coupling means classes interact through abstract interfaces rather than concrete implementations, allowing components to be replaced or tested in isolation.

---

### 6.3 The Law of Demeter (Principle of Least Knowledge)

> *"Talk only to your immediate friends; do not speak to strangers."*

A method `m` of an object `O` should only invoke methods on:
1. `O` itself.
2. Parameters passed directly into `m`.
3. Any objects instantiated directly within `m`.
4. Direct instance variables of `O`.

```python
# VIOLATION: Train-wreck call reaching 3 layers deep into internal state
city = order.get_customer().get_address().get_city()

# CLEAN: Delegate behavior to the immediate collaborator
city = order.get_delivery_city()
```

Reaching through chains of getters tightly couples the caller to the internal object topology of three separate classes. If `Address` is refactored, the caller breaks.

---

### 6.4 Tell, Don't Ask

> *"Tell objects what you want them to do; do not interrogate their internal state and perform work on their behalf."*

```python
# ANTI-PATTERN ("Ask"): Interrogate state and calculate externally
if account.get_balance() >= amount:
    new_balance = account.get_balance() - amount
    account.set_balance(new_balance)
    dispense_cash()

# CLEAN PATTERN ("Tell"): Command the object to execute its own logic
if account.withdraw(amount):
    dispense_cash()
```

When callers query state and manipulate it externally, business logic duplicates across multiple callers and invariants are easily violated. The "Tell, Don't Ask" principle keeps logic located alongside the data it operates upon.
