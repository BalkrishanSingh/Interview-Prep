# Module 02: OOP — Creational Design Patterns

---

## 1. Overview of Creational Patterns

Creational design patterns abstract the instantiation process. They decouple a system from how its objects are created, composed, and represented.

| Pattern | Primary Intent | Key Distinction |
| :--- | :--- | :--- |
| **Singleton** | Ensures exactly one instance of a class exists globally. | State and coordination manager (e.g., config, thread pool). |
| **Factory Method** | Defines an interface for object creation, deferring choice to subclasses. | Creates **one** product via inheritance. |
| **Abstract Factory** | Creates **families** of related or dependent objects. | Factory of factories; creates multiple related products. |
| **Builder** | Constructs complex objects step-by-step. | Separates construction process from representation. |
| **Prototype** | Clones existing objects rather than instantiating from scratch. | Creates duplicates via shallow/deep copy. |

---

## 2. Singleton Pattern

### 2.1 Thread-Safe Double-Checked Locking Pattern (Python)
In multi-threaded environments, two threads checking `_instance is None` simultaneously can create two distinct objects. The **Double-Checked Locking** idiom prevents this race condition efficiently.

```python
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        # First check (unlocked) for high-performance reading once initialized
        if cls._instance is None:
            with cls._lock:
                # Second check (locked) to prevent concurrent thread creation
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Verification:
s1 = ThreadSafeSingleton()
s2 = ThreadSafeSingleton()
assert s1 is s2, "Instances must be memory-identical"
```

### 2.2 Pythonic Metaclass Implementation
The cleanest and most reusable way to implement Singletons across multiple classes in Python:

```python
class SingletonMeta(type):
    """Metaclass that creates a Singleton base."""
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__(*args, **kwargs)
                cls._instances[cls] = instance
        return cls._instances[cls]

class DatabaseConnectionPool(metaclass=SingletonMeta):
    def __init__(self, connection_str: str = "postgres://main-db:5432"):
        self.connection_str = connection_str
        print("Initializing heavy database connection pool...")

pool1 = DatabaseConnectionPool()
pool2 = DatabaseConnectionPool()
assert pool1 is pool2  # Initializer runs only once!
```

---

## 3. Factory Method Pattern

Decouples the client from concrete instantiation logic by delegating creation to dedicated factory subclasses or creator methods.

```python
from abc import ABC, abstractmethod

# Product Interface
class Notification(ABC):
    @abstractmethod
    def notify(self, message: str) -> None: pass

# Concrete Products
class EmailNotification(Notification):
    def notify(self, message: str) -> None:
        print(f"[EMAIL] {message}")

class SMSNotification(Notification):
    def notify(self, message: str) -> None:
        print(f"[SMS] {message}")

class PushNotification(Notification):
    def notify(self, message: str) -> None:
        print(f"[PUSH] {message}")

# Creator Factory
class NotificationFactory:
    @staticmethod
    def create_notification(channel: str) -> Notification:
        channels = {
            "email": EmailNotification,
            "sms": SMSNotification,
            "push": PushNotification
        }
        creator = channels.get(channel.lower())
        if not creator:
            raise ValueError(f"Unknown notification channel: {channel}")
        return creator()

# Client usage:
notifier = NotificationFactory.create_notification("sms")
notifier.notify("Your OTP is 482910")
```

---

## 4. Abstract Factory Pattern

Provides an interface for creating **families** of related objects without specifying their concrete classes.

### Scenario: Multi-Platform UI Framework (Dark vs. Light Theme)
```python
# Abstract Products
class Button(ABC):
    @abstractmethod
    def render(self) -> str: pass

class Checkbox(ABC):
    @abstractmethod
    def toggle(self) -> str: pass

# Concrete Dark Products
class DarkButton(Button):
    def render(self) -> str: return "[Dark Slate Button]"

class DarkCheckbox(Checkbox):
    def toggle(self) -> str: return "[Dark Neon Checkbox: Checked]"

# Concrete Light Products
class LightButton(Button):
    def render(self) -> str: return "[Light White Button]"

class LightCheckbox(Checkbox):
    def toggle(self) -> str: return "[Light Grey Checkbox: Checked]"

# Abstract Factory
class UIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: pass
    @abstractmethod
    def create_checkbox(self) -> Checkbox: pass

# Concrete Factories
class DarkThemeFactory(UIFactory):
    def create_button(self) -> Button: return DarkButton()
    def create_checkbox(self) -> Checkbox: return DarkCheckbox()

class LightThemeFactory(UIFactory):
    def create_button(self) -> Button: return LightButton()
    def create_checkbox(self) -> Checkbox: return LightCheckbox()

# Client Application
def render_dashboard(factory: UIFactory):
    btn = factory.create_button()
    chk = factory.create_checkbox()
    print(f"Rendered: {btn.render()} and {chk.toggle()}")

# Seamless theme switching:
render_dashboard(DarkThemeFactory())
render_dashboard(LightThemeFactory())
```

---

## 5. Builder Pattern

Constructs complex objects incrementally. Allows producing different types and representations of an object using the same construction code.

```python
class HTTPRequest:
    def __init__(self):
        self.url = None
        self.method = "GET"
        self.headers = {}
        self.params = {}
        self.body = None

    def __repr__(self):
        return f"HTTPRequest(method={self.method}, url={self.url}, headers={self.headers}, body={self.body})"

class HTTPRequestBuilder:
    def __init__(self, url: str):
        self._request = HTTPRequest()
        self._request.url = url

    def set_method(self, method: str):
        self._request.method = method.upper()
        return self  # Return self to allow method chaining (Fluent Interface)

    def add_header(self, key: str, value: str):
        self._request.headers[key] = value
        return self

    def set_json_body(self, payload: str):
        self._request.body = payload
        self._request.headers["Content-Type"] = "application/json"
        return self

    def build(self) -> HTTPRequest:
        if not self._request.url:
            raise ValueError("URL must be specified")
        return self._request

# Fluent construction:
request = (HTTPRequestBuilder("https://api.example.com/v1/auth")
           .set_method("POST")
           .add_header("Authorization", "Bearer token_xyz")
           .set_json_body('{"user": "alice"}')
           .build())
print(request)
```

---

## 6. Prototype Pattern

Allows cloning existing objects without making code dependent on their concrete classes.

```python
import copy

class NetworkConfigPrototype:
    def __init__(self, subnet: str, dns_servers: list[str], security_rules: dict):
        self.subnet = subnet
        self.dns_servers = dns_servers
        self.security_rules = security_rules

    def clone(self):
        """Creates a complete deep copy of the object state."""
        return copy.deepcopy(self)

base_config = NetworkConfigPrototype(
    subnet="192.168.1.0/24",
    dns_servers=["8.8.8.8", "1.1.1.1"],
    security_rules={"inbound": ["ALLOW 80", "ALLOW 443"]}
)

# Clone and modify without affecting the original prototype:
prod_config = base_config.clone()
prod_config.subnet = "10.0.0.0/16"
prod_config.security_rules["inbound"].append("ALLOW 22")

assert base_config.subnet == "192.168.1.0/24"  # Unaffected!
```
