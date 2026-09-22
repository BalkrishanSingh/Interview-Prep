# Module 02: OOP — SOLID Principles In-Depth

---

## 1. Single Responsibility Principle (SRP)

> **"A class should have one, and only one, reason to change."**

### 1.1 The Anti-Pattern (Violation)
A "God Object" or class that handles multiple orthogonal concerns: business logic, persistence, and external notifications.

```python
# VIOLATION: Has 3 reasons to change (auth logic, DB schema, email service)
class UserRegistrationService:
    def register_user(self, username: str, email: str, raw_pwd: str):
        # 1. Validation & Hashing
        if len(raw_pwd) < 8:
            raise ValueError("Password too short")
        hashed_pwd = hash(raw_pwd)

        # 2. Database Persistence
        db_connection = "mysql://localhost:3306"
        print(f"Connecting to {db_connection} and saving {username}")

        # 3. Email Notification
        smtp_server = "smtp.mailgun.org"
        print(f"Connecting to {smtp_server} and emailing {email}")
```

### 1.2 The Clean Refactoring
Split into specialized, single-purpose classes:

```python
class PasswordService:
    def hash_password(self, raw_pwd: str) -> str:
        if len(raw_pwd) < 8:
            raise ValueError("Password must be at least 8 characters")
        return f"hashed_{raw_pwd}"

class UserRepository:
    def save(self, username: str, email: str, hashed_pwd: str) -> None:
        print(f"INSERT INTO users (username, email, pwd) VALUES ('{username}', ...)")

class EmailNotificationService:
    def send_welcome_email(self, email: str) -> None:
        print(f"Dispatching welcome message to {email}")

class UserRegistrationService:
    """Coordinates registration without knowing DB or SMTP implementation details."""
    def __init__(self, pwd_svc: PasswordService, repo: UserRepository, email_svc: EmailNotificationService):
        self.pwd_svc = pwd_svc
        self.repo = repo
        self.email_svc = email_svc

    def register(self, username: str, email: str, raw_pwd: str) -> None:
        hashed = self.pwd_svc.hash_password(raw_pwd)
        self.repo.save(username, email, hashed)
        self.email_svc.send_welcome_email(email)
```

---

## 2. Open/Closed Principle (OCP)

> **"Software entities (classes, modules, functions) should be open for extension, but closed for modification."**

### 2.1 The Anti-Pattern (Violation)
Modifying existing code and adding `if/elif` branches whenever a new business rule or payment type is introduced.

```python
# VIOLATION: Adding a new payment type requires modifying this function!
class PaymentProcessor:
    def process_payment(self, payment_type: str, amount: float):
        if payment_type == "credit_card":
            print(f"Charging ${amount} to Credit Card")
        elif payment_type == "paypal":
            print(f"Redirecting ${amount} to PayPal API")
        elif payment_type == "crypto":  # Added later: had to modify existing tested code!
            print(f"Broadcasting ${amount} to Blockchain")
        else:
            raise ValueError("Unsupported payment method")
```

### 2.2 The Clean Refactoring (Strategy / Polymorphism)
Define an open interface. Add new payment types by creating new classes without altering existing code:

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None: pass

class CreditCardPayment(PaymentStrategy):
    def pay(self, amount: float) -> None:
        print(f"Processing ${amount} via Credit Card Gateway")

class PayPalPayment(PaymentStrategy):
    def pay(self, amount: float) -> None:
        print(f"Processing ${amount} via PayPal API")

class CryptoPayment(PaymentStrategy):
    def pay(self, amount: float) -> None:
        print(f"Broadcasting ${amount} transaction to Blockchain")

class CheckoutService:
    """Closed for modification: works with ANY future PaymentStrategy."""
    def execute_transaction(self, strategy: PaymentStrategy, amount: float) -> None:
        strategy.pay(amount)
```

---

## 3. Liskov Substitution Principle (LSP)

> **"Subtypes must be substitutable for their base types without altering the correctness of the program."**

### 3.1 The Canonical Classic Violation (Rectangle & Square)
A mathematical square is a rectangle with equal sides. However, in software design, modeling `Square` as a subclass of `Rectangle` violates behavioral subtyping!

```python
# VIOLATION: Square breaks the invariant of Rectangle
class Rectangle:
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height

    def set_width(self, width: float): self._width = width
    def set_height(self, height: float): self._height = height
    def area(self) -> float: return self._width * self._height

class Square(Rectangle):
    def set_width(self, width: float):
        self._width = width
        self._height = width  # Side effect! Modifies height too

    def set_height(self, height: float):
        self._width = height
        self._height = height

def verify_rectangle_area(rect: Rectangle):
    rect.set_width(5)
    rect.set_height(4)
    # For a genuine Rectangle, area MUST be 5 * 4 = 20.
    # But if rect is a Square, area will be 4 * 4 = 16! Test fails!
    assert rect.area() == 20, f"Expected 20, got {rect.area()}"
```

### 3.2 The Clean Refactoring
Separate them under a common abstract geometry interface:

```python
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

class Square(Shape):
    def __init__(self, side: float):
        self.side = side

    def area(self) -> float:
        return self.side * self.side
```

---

## 4. Interface Segregation Principle (ISP)

> **"Clients should not be forced to depend upon interfaces that they do not use."**

### 4.1 The Anti-Pattern (Violation: Fat Interface)
```python
# VIOLATION: Forcing simple devices to implement methods they don't support
class SmartMachine(ABC):
    @abstractmethod
    def print_doc(self, doc: str): pass

    @abstractmethod
    def scan_doc(self) -> str: pass

    @abstractmethod
    def fax_doc(self, doc: str): pass

class BasicHomePrinter(SmartMachine):
    def print_doc(self, doc: str):
        print(f"Printing: {doc}")

    def scan_doc(self) -> str:
        raise NotImplementedError("This printer has no scanner!")

    def fax_doc(self, doc: str):
        raise NotImplementedError("This printer cannot fax!")
```

### 4.2 The Clean Refactoring
Segregate into granular, cohesive interfaces:

```python
class Printer(ABC):
    @abstractmethod
    def print_doc(self, doc: str): pass

class Scanner(ABC):
    @abstractmethod
    def scan_doc(self) -> str: pass

class FaxMachine(ABC):
    @abstractmethod
    def fax_doc(self, doc: str): pass

# Simple printer only implements Printer
class BasicHomePrinter(Printer):
    def print_doc(self, doc: str):
        print(f"Printing: {doc}")

# High-end enterprise machine implements multiple interfaces
class AllInOneOfficePrinter(Printer, Scanner, FaxMachine):
    def print_doc(self, doc: str): print(f"Printing: {doc}")
    def scan_doc(self) -> str: return "Scanned document content"
    def fax_doc(self, doc: str): print(f"Faxing: {doc}")
```

---

## 5. Dependency Inversion Principle (DIP)

> **"High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions."**

### 5.1 The Anti-Pattern (Violation)
A high-level business service directly instantiating a concrete low-level infrastructure class (tight coupling).

```python
# Low-level infrastructure
class MySQLDatabase:
    def execute_query(self, sql: str):
        print(f"Executing '{sql}' on MySQL engine.")

# High-level business logic
class OrderService:
    def __init__(self):
        # VIOLATION: Hard-coded dependency on concrete MySQLDatabase!
        self.db = MySQLDatabase()

    def create_order(self, order_id: int):
        self.db.execute_query(f"INSERT INTO orders VALUES ({order_id})")
```

### 5.2 The Clean Refactoring (Dependency Injection)
```python
# Abstraction
class Database(ABC):
    @abstractmethod
    def execute_query(self, sql: str) -> None: pass

# Low-level concrete implementations depend on the abstraction
class MySQLDatabase(Database):
    def execute_query(self, sql: str) -> None:
        print(f"Executing '{sql}' on MySQL.")

class PostgreSQLDatabase(Database):
    def execute_query(self, sql: str) -> None:
        print(f"Executing '{sql}' on Postgres.")

# High-level module depends on the abstraction
class OrderService:
    def __init__(self, db: Database):  # Injected via constructor
        self.db = db

    def create_order(self, order_id: int) -> None:
        self.db.execute_query(f"INSERT INTO orders VALUES ({order_id})")

# Inversion of Control: The caller decides which DB engine to inject!
order_service = OrderService(PostgreSQLDatabase())
order_service.create_order(101)
```
