# Module 02: OOP — Core Pillars & Object-Oriented Mental Models

---

## 1. The Four Foundational Pillars of OOP

Object-Oriented Programming (OOP) organizes software design around **objects** (encapsulating state and behavior) rather than standalone functions and procedural logic.

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
 │   & Bundling │        │  Interfaces │ │  & "Is-A"   │    │  & Overrides │
 └──────────────┘        └─────────────┘ └─────────────┘    └──────────────┘
```

---

### 1.1 Encapsulation
- **Definition**: Bundling data attributes and the methods that mutate that data within a unified class structure, while restricting direct external access to prevent arbitrary modification.
- **Why it matters**: Enforces class invariants, hides implementation details, and reduces coupling between callers and class internals.
- **Python Access Control Conventions**:
  - `public_var`: Accessible anywhere.
  - `_protected_var`: Internal by convention (hints to developers: do not access directly outside the class or subclasses).
  - `__private_var`: Triggers name mangling (`_ClassName__private_var`) to prevent accidental overrides in subclasses.

```python
class BankAccount:
    """Demonstrates Encapsulation via Python @property decorators."""
    def __init__(self, account_holder: str, initial_deposit: float):
        self.holder = account_holder
        self.__balance = initial_deposit  # Private variable

    @property
    def balance(self) -> float:
        """Getter for controlled read access."""
        return self.__balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit amount must be strictly positive.")
        self.__balance += amount

    def withdraw(self, amount: float) -> bool:
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
```

---

### 1.2 Abstraction
- **Definition**: Exposing only the essential interface to the outside world while hiding internal implementation complexity.
- **Analogy**: A smartphone touchscreen. Users tap an icon to send an email; they do not construct TCP packets or write bytes to radio transmitters.
- **Implementation**: Abstract Base Classes using Python's standard `abc` module (`ABC`, `@abstractmethod`).

```python
from abc import ABC, abstractmethod

class NotificationService(ABC):
    """Abstract interface defining the contract for sending notifications."""
    
    @abstractmethod
    def send(self, recipient: str, message: str) -> bool:
        """Dispatches notification. Must be implemented by concrete subclasses."""
        pass

class EmailNotification(NotificationService):
    def send(self, recipient: str, message: str) -> bool:
        # Handles SMTP connection, TLS handshake, email headers
        print(f"Sending SMTP email to {recipient}: {message}")
        return True

class SMSNotification(NotificationService):
    def send(self, recipient: str, message: str) -> bool:
        # Connects to Twilio / Telecom SMS gateway API
        print(f"Dispatching SMS to {recipient}: {message}")
        return True
```

---

### 1.3 Inheritance
- **Definition**: The mechanism where a new class (subclass/child) derives attributes and behaviors from an existing class (superclass/parent), establishing an **"is-a"** relationship.
- **Multiple Inheritance & MRO (Method Resolution Order)**:
  - Python natively supports multiple inheritance. When multiple parents share methods (the Diamond Problem), Python resolves method calls using the **C3 Linearization Algorithm**.
  - Check resolution order dynamically via `ClassName.__mro__`.

```python
class A:
    def ping(self): return "A"

class B(A):
    def ping(self): return "B"

class C(A):
    def ping(self): return "C"

class D(B, C):
    pass

# MRO order: D -> B -> C -> A -> object
print([cls.__name__ for cls in D.__mro__])  # ['D', 'B', 'C', 'A', 'object']
```

---

### 1.4 Polymorphism ("Many Forms")
- **Definition**: The ability of different classes to respond to the same interface or method call with class-specific behavior.

| Polymorphism Type | Mechanism | Implementation Details |
| :--- | :--- | :--- |
| **Static / Compile-Time** | Method Overloading | In Java/C++, defining multiple methods with the same name but different argument counts/types. In Python, achieved using optional arguments, `*args/**kwargs`, or `functools.singledispatch`. |
| **Dynamic / Runtime** | Method Overriding | Subclasses override a method defined in the base class. Resolved dynamically at runtime via virtual method tables (VTable). |

```python
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: pass

class Circle(Shape):
    def __init__(self, r: float): self.r = r
    def area(self) -> float: return 3.14159 * self.r * self.r

class Rectangle(Shape):
    def __init__(self, w: float, h: float): self.w, self.h = w, h
    def area(self) -> float: return self.w * self.h

def calculate_all_areas(shapes: list[Shape]) -> list[float]:
    # Polymorphic dispatch: shape.area() invokes correct implementation
    return [shape.area() for shape in shapes]
```

---

## 2. Abstract Class vs. Interface

| Dimension | Abstract Class | Interface |
| :--- | :--- | :--- |
| **State / Variables** | Can contain instance state (attributes, instance fields) | Pure contract. Cannot hold instance state. |
| **Implementation** | Can supply both abstract declarations AND concrete method implementations | Pure declarations (prior to Java 8 default methods). |
| **Inheritance Model** | Single inheritance in Java/C# (Multiple in Python via ABC) | A class can implement multiple interfaces. |
| **Design Intent** | "Is-A" relationship sharing identity and base code | "Can-Do" capability contract across unrelated classes |

---

## 3. Composition vs. Inheritance

> [!IMPORTANT]
> **Foundational Design Heuristic**: *"Favor Object Composition over Class Inheritance."*

- **Inheritance ("Is-A")**: Creates tight coupling. Fragile base class problem: modifying a base class breaks descendant classes.
- **Composition ("Has-A")**: Objects contain references to other objects and delegate tasks. Highly modular, testable, and swappable at runtime.

```python
# Composition in practice
class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool: pass

class StripeProcessor(PaymentProcessor):
    def pay(self, amount: float) -> bool:
        print(f"Processing ${amount} via Stripe API.")
        return True

class PayPalProcessor(PaymentProcessor):
    def pay(self, amount: float) -> bool:
        print(f"Processing ${amount} via PayPal API.")
        return True

class CheckoutService:
    def __init__(self, processor: PaymentProcessor):
        self.processor = processor  # Composition: Has-A payment processor

    def complete_order(self, amount: float) -> None:
        self.processor.pay(amount)

# Processors can be swapped dynamically without altering CheckoutService:
checkout = CheckoutService(StripeProcessor())
checkout.complete_order(99.0)
```

---

## 4. Python Specific OOP Essentials

### 4.1 Instance vs. Class vs. Static Methods
```python
class DatabaseConnection:
    default_timeout = 30  # Class attribute

    def __init__(self, connection_str: str):
        self.connection_str = connection_str  # Instance attribute

    # 1. Instance Method: Receives 'self' (interacts with instance state)
    def connect(self) -> None:
        print(f"Connecting to {self.connection_str}...")

    # 2. Class Method: Receives 'cls' (factory constructor, modifies class state)
    @classmethod
    def create_local_dev(cls):
        return cls("localhost:5432/dev_db")

    # 3. Static Method: Receives neither 'self' nor 'cls' (pure isolated logic)
    @staticmethod
    def is_valid_port(port: int) -> bool:
        return 1 <= port <= 65535
```

### 4.2 Core Magic / Dunder Methods
- `__init__(self, ...)`: Instance constructor / initializer.
- `__repr__(self)`: Formal string representation for developers (`eval(repr(obj)) == obj`).
- `__str__(self)`: Informal user-facing representation for `print()`.
- `__eq__(self, other)`: Equality operator `==`.
- `__call__(self, ...)`: Enables instances to be called directly as functions: `obj()`.
