# Module 02: OOP — Structural Design Patterns

---

## 1. Overview of Structural Patterns

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

| Pattern | Primary Intent | Key Distinction |
| :--- | :--- | :--- |
| **Adapter** | Converts the interface of a class into another interface clients expect. | Bridges two incompatible interfaces. |
| **Decorator** | Attaches additional responsibilities to an object dynamically. | Flexible alternative to subclassing for extending functionality. |
| **Facade** | Provides a simplified, unified high-level interface to a complex subsystem. | Simplifies an entire subsystem. |
| **Proxy** | Provides a surrogate or placeholder for another object to control access to it. | Controls, caches, or intercepts access to an object. |
| **Composite** | Composes objects into tree structures to represent part-whole hierarchies. | Allows treating individual objects and compositions uniformly. |

---

## 2. Adapter Pattern

### Scenario: Integrating a Third-Party JSON Analytics API with Legacy XML Client
```python
from abc import ABC, abstractmethod

# Existing Client Interface expects XML
class XMLDataProvider(ABC):
    @abstractmethod
    def get_data_xml(self) -> str: pass

# New Third-Party Service returns JSON (Incompatible!)
class ModernJSONAnalyticsService:
    def fetch_analytics_json(self) -> dict:
        return {"active_users": 1420, "revenue": 89400.50}

# The Adapter bridges ModernJSONAnalyticsService to XMLDataProvider
class AnalyticsAdapter(XMLDataProvider):
    def __init__(self, modern_service: ModernJSONAnalyticsService):
        self.modern_service = modern_service

    def get_data_xml(self) -> str:
        data = self.modern_service.fetch_analytics_json()
        # Converts dictionary to expected XML format
        xml_output = (f"<analytics>"
                      f"<active_users>{data['active_users']}</active_users>"
                      f"<revenue>{data['revenue']}</revenue>"
                      f"</analytics>")
        return xml_output

# Client function only accepts XMLDataProvider
def generate_report(provider: XMLDataProvider):
    print("Report XML Content:\n", provider.get_data_xml())

adapter = AnalyticsAdapter(ModernJSONAnalyticsService())
generate_report(adapter)
```

---

## 3. Decorator Pattern

Dynamically adds new behavior to objects without modifying their source code or using subclass inheritance (preventing class explosion).

```python
from abc import ABC, abstractmethod

# Component Interface
class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float: pass
    @abstractmethod
    def description(self) -> str: pass

# Concrete Component
class SimpleCoffee(Coffee):
    def cost(self) -> float: return 2.00
    def description(self) -> str: return "Simple Coffee"

# Base Decorator
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._decorated_coffee = coffee

    def cost(self) -> float:
        return self._decorated_coffee.cost()

    def description(self) -> str:
        return self._decorated_coffee.description()

# Concrete Decorators
class MilkDecorator(CoffeeDecorator):
    def cost(self) -> float: return self._decorated_coffee.cost() + 0.50
    def description(self) -> str: return f"{self._decorated_coffee.description()}, Milk"

class VanillaDecorator(CoffeeDecorator):
    def cost(self) -> float: return self._decorated_coffee.cost() + 0.75
    def description(self) -> str: return f"{self._decorated_coffee.description()}, Vanilla"

# Dynamic composition at runtime:
my_coffee = SimpleCoffee()
my_coffee = MilkDecorator(my_coffee)
my_coffee = VanillaDecorator(my_coffee)

print(f"{my_coffee.description()} -> ${my_coffee.cost():.2f}")
# Output: Simple Coffee, Milk, Vanilla -> $3.25
```

---

## 4. Facade Pattern

Provides a simple, unified interface to a complex library, framework, or set of interacting subsystems.

```python
# Subsystem 1
class InventoryService:
    def check_stock(self, item_id: str) -> bool:
        print(f"[Inventory] Item {item_id} is in stock.")
        return True

# Subsystem 2
class PaymentGateway:
    def charge(self, amount: float) -> bool:
        print(f"[Payment] Charged ${amount} successfully.")
        return True

# Subsystem 3
class LogisticsService:
    def create_shipping_label(self, item_id: str, address: str):
        print(f"[Logistics] Shipping label generated for {address}.")

# Facade
class OrderProcessingFacade:
    def __init__(self):
        self.inventory = InventoryService()
        self.payment = PaymentGateway()
        self.logistics = LogisticsService()

    def place_order(self, item_id: str, amount: float, shipping_address: str) -> bool:
        """Single method coordinates the entire distributed order workflow."""
        if not self.inventory.check_stock(item_id):
            return False
        if not self.payment.charge(amount):
            return False
        self.logistics.create_shipping_label(item_id, shipping_address)
        print("[Order Facade] Order placed successfully!")
        return True

# Client interacts with 1 clean method instead of 3 subsystems:
facade = OrderProcessingFacade()
facade.place_order("LAPTOP-99", 1200.0, "123 Electronic City, Bangalore")
```

---

## 5. Proxy Pattern (Virtual & Protection Proxies)

A proxy controls access to the original object, enabling lazy loading, caching, authorization, or logging.

### 5.1 Virtual Proxy (Lazy Loading Heavy Resources)
```python
class RealHeavyDatabaseReport:
    def __init__(self, filename: str):
        self.filename = filename
        self._load_gigabytes_from_disk()

    def _load_gigabytes_from_disk(self):
        print(f"Loading 10 GB dataset from {self.filename}... (Takes 5 seconds)")

    def display(self):
        print(f"Displaying visualizations for {self.filename}")

class LazyReportProxy:
    """Delays instantiating the heavy object until display() is called."""
    def __init__(self, filename: str):
        self.filename = filename
        self._real_report = None

    def display(self):
        if self._real_report is None:
            self._real_report = RealHeavyDatabaseReport(self.filename)
        self._real_report.display()

# Instantiation is instantaneous! Heavy loading happens only on demand:
proxy = LazyReportProxy("annual_financials_2024.parquet")
proxy.display()
```

### 5.2 Protection Proxy (Role-Based Access Control)
```python
class SecureDocumentService:
    def delete_records(self):
        print("CRITICAL: Database records deleted permanently.")

class DocumentSecurityProxy:
    def __init__(self, service: SecureDocumentService, user_role: str):
        self._service = service
        self.user_role = user_role

    def delete_records(self):
        if self.user_role.upper() != "ADMIN":
            raise PermissionError("Access Denied: Requires ADMIN privilege.")
        self._service.delete_records()
```

---

## 6. Composite Pattern

Treats individual objects (leaves) and compositions of objects (containers) uniformly using a shared component interface.

```python
class FileSystemItem(ABC):
    @abstractmethod
    def get_size(self) -> int: pass
    @abstractmethod
    def ls(self, indent: int = 0) -> None: pass

# Leaf
class File(FileSystemItem):
    def __init__(self, name: str, size: int):
        self.name = name
        self.size = size

    def get_size(self) -> int: return self.size
    def ls(self, indent: int = 0) -> None:
        print("  " * indent + f"- {self.name} ({self.size} KB)")

# Composite
class Directory(FileSystemItem):
    def __init__(self, name: str):
        self.name = name
        self.children: list[FileSystemItem] = []

    def add(self, item: FileSystemItem):
        self.children.append(item)

    def get_size(self) -> int:
        return sum(child.get_size() for child in self.children)

    def ls(self, indent: int = 0) -> None:
        print("  " * indent + f"+ [{self.name}]")
        for child in self.children:
            child.ls(indent + 1)

# Building a hierarchy:
root = Directory("root")
home = Directory("home")
home.add(File("resume.pdf", 120))
home.add(File("photo.jpg", 2500))
root.add(home)
root.add(File("config.sys", 10))

root.ls()
print(f"Total Size: {root.get_size()} KB")
```
