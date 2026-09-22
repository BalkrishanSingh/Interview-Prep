# Module 02: OOP — Behavioral Design Patterns

---

## 1. Overview of Behavioral Patterns

Behavioral patterns focus on algorithms, responsibility assignment, and communication protocols between interacting objects.

| Pattern | Primary Intent | Key Distinction |
| :--- | :--- | :--- |
| **Observer** | Notifies multiple subscribers automatically of any state change in a subject. | 1-to-Many event notification. |
| **Strategy** | Encapsulates family of algorithms and makes them interchangeable at runtime. | Alters how an object accomplishes a task. |
| **Command** | Encapsulates a request as a standalone object, enabling Undo/Redo and queues. | Parameterizes clients with different requests. |
| **State** | Alters object behavior when its internal state changes, avoiding massive conditionals. | Replaces complex state `if/elif` ladders with polymorphism. |
| **Template Method** | Defines algorithm skeleton in a superclass, deferring step details to subclasses. | Invariant workflow structure with customizable steps. |

---

## 2. Observer Pattern (Publish-Subscribe)

The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. It forms the backbone of event-driven architectures and reactive UI models.

```python
from abc import ABC, abstractmethod

# Observer Interface
class Subscriber(ABC):
    @abstractmethod
    def update(self, channel_name: str, video_title: str) -> None: pass

# Concrete Observers
class MobileAppUser(Subscriber):
    def __init__(self, username: str):
        self.username = username

    def update(self, channel_name: str, video_title: str) -> None:
        print(f"Push to {self.username}: '{channel_name}' uploaded '{video_title}'!")

class EmailSubscriber(Subscriber):
    def __init__(self, email: str):
        self.email = email

    def update(self, channel_name: str, video_title: str) -> None:
        print(f"Emailing {self.email}: New video available: {video_title}")

# Subject / Publisher
class YouTubeChannel:
    def __init__(self, name: str):
        self.name = name
        self._subscribers: list[Subscriber] = []

    def subscribe(self, subscriber: Subscriber) -> None:
        self._subscribers.append(subscriber)

    def unsubscribe(self, subscriber: Subscriber) -> None:
        self._subscribers.remove(subscriber)

    def upload_video(self, title: str) -> None:
        print(f"\n--- {self.name} uploaded: '{title}' ---")
        for sub in self._subscribers:
            sub.update(self.name, title)

# Usage
channel = YouTubeChannel("TechArchitectureChannel")
alice = MobileAppUser("alice_99")
bob = EmailSubscriber("bob@example.com")

channel.subscribe(alice)
channel.subscribe(bob)
channel.upload_video("Understanding Distributed Event Queues")
```

---

## 3. Strategy Pattern

Encapsulates interchangeable algorithms behind a common interface.

### Scenario: Navigation Route Calculation
```python
class RoutingStrategy(ABC):
    @abstractmethod
    def build_route(self, origin: str, destination: str) -> str: pass

class DrivingStrategy(RoutingStrategy):
    def build_route(self, origin: str, destination: str) -> str:
        return f"Highway route from {origin} to {destination} (Duration: 35 mins)"

class WalkingStrategy(RoutingStrategy):
    def build_route(self, origin: str, destination: str) -> str:
        return f"Pedestrian walkways from {origin} to {destination} (Duration: 2 hrs)"

class TransitStrategy(RoutingStrategy):
    def build_route(self, origin: str, destination: str) -> str:
        return f"Metro Line 2 from {origin} to {destination} (Duration: 45 mins)"

class Navigator:
    def __init__(self, strategy: RoutingStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: RoutingStrategy):
        self._strategy = strategy

    def navigate(self, start: str, end: str) -> None:
        print(self._strategy.build_route(start, end))

# Seamless runtime switching:
nav = Navigator(DrivingStrategy())
nav.navigate("Indiranagar", "Electronic City")

nav.set_strategy(TransitStrategy())
nav.navigate("Indiranagar", "Electronic City")
```

---

## 4. Command Pattern (with Undo / Redo)

Encapsulates all details of an intent into a command object. This decouples the object invoking the action from the object executing it.

```python
# Command Interface
class Command(ABC):
    @abstractmethod
    def execute(self) -> None: pass
    @abstractmethod
    def undo(self) -> None: pass

# Receiver (Text Editor Buffer)
class TextDocument:
    def __init__(self):
        self.text = ""

    def append(self, text_to_add: str):
        self.text += text_to_add

    def delete_last(self, length: int):
        self.text = self.text[:-length]

# Concrete Command
class WriteCommand(Command):
    def __init__(self, document: TextDocument, text: str):
        self.document = document
        self.text = text

    def execute(self):
        self.document.append(self.text)

    def undo(self):
        self.document.delete_last(len(self.text))

# Invoker (Tracks History)
class EditorHistory:
    def __init__(self):
        self._history: list[Command] = []

    def execute_command(self, cmd: Command):
        cmd.execute()
        self._history.append(cmd)

    def undo_last(self):
        if self._history:
            cmd = self._history.pop()
            cmd.undo()

# Test Undo Execution
doc = TextDocument()
editor = EditorHistory()

editor.execute_command(WriteCommand(doc, "Hello "))
editor.execute_command(WriteCommand(doc, "World!"))
print("Current Text:", doc.text)  # "Hello World!"

editor.undo_last()
print("After Undo:", doc.text)    # "Hello "
```

---

## 5. State Pattern

Allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

### Scenario: Order Lifecycle (New $\to$ Paid $\to$ Shipped)
```python
# State Interface
class OrderState(ABC):
    @abstractmethod
    def pay(self, order: "Order") -> None: pass
    @abstractmethod
    def ship(self, order: "Order") -> None: pass

# Concrete State 1: New
class NewOrderState(OrderState):
    def pay(self, order: "Order") -> None:
        print("Payment successful. Transitioning to PAID state.")
        order.set_state(PaidOrderState())

    def ship(self, order: "Order") -> None:
        print("Cannot ship an unpaid order!")

# Concrete State 2: Paid
class PaidOrderState(OrderState):
    def pay(self, order: "Order") -> None:
        print("Order is already paid.")

    def ship(self, order: "Order") -> None:
        print("Dispatching order. Transitioning to SHIPPED state.")
        order.set_state(ShippedOrderState())

# Concrete State 3: Shipped
class ShippedOrderState(OrderState):
    def pay(self, order: "Order") -> None:
        print("Order is already paid and shipped.")

    def ship(self, order: "Order") -> None:
        print("Order has already been shipped!")

# Context
class Order:
    def __init__(self):
        self._state: OrderState = NewOrderState()

    def set_state(self, state: OrderState):
        self._state = state

    def pay(self): self._state.pay(self)
    def ship(self): self._state.ship(self)

order = Order()
order.ship()  # Fails: Cannot ship an unpaid order!
order.pay()   # Succeeds -> Transitions to Paid
order.ship()  # Succeeds -> Transitions to Shipped
```

---

## 6. Template Method Pattern

Defines the skeleton of an algorithm in a base class, allowing subclasses to override specific steps without modifying the overall workflow.

```python
class DataPipeline(ABC):
    def run_pipeline(self):
        """Template Method defines fixed sequential execution order."""
        self.extract_data()
        self.transform_data()
        self.load_data()
        print("Pipeline execution completed successfully!\n")

    @abstractmethod
    def extract_data(self): pass

    @abstractmethod
    def transform_data(self): pass

    def load_data(self):
        # Default step common to all pipelines
        print("Loading processed rows into Enterprise Data Lake.")

class CSVDataPipeline(DataPipeline):
    def extract_data(self):
        print("Extracting records from sales_data.csv...")

    def transform_data(self):
        print("Cleaning missing values and casting numerical columns.")

class APIDataPipeline(DataPipeline):
    def extract_data(self):
        print("Polling JSON endpoints with OAuth2 token...")

    def transform_data(self):
        print("Normalizing JSON payload into relational format.")

# Invariant workflow preserved:
CSVDataPipeline().run_pipeline()
APIDataPipeline().run_pipeline()
```
