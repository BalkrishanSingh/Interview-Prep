# Module 02: OOP — Low-Level Design (LLD) Case Studies

---

## 1. Low-Level Design Methodology

Object-oriented low-level design structures a software component from requirements to concrete code through a 4-step framework:
1. **Clarify Requirements & Scoping**: Identify constraints, entities, operational capacity, and edge cases.
2. **Identify Core Entities & Enums**: Extract domain nouns from requirements (`Vehicle`, `Spot`, `Ticket`, `Floor`).
3. **Define Class Relationships**: Establish Inheritance ("Is-A") vs. Composition ("Has-A") boundaries.
4. **Apply Appropriate Design Patterns**: Use Singleton for stateful controllers, Strategy for dynamic algorithms, and Factory for object creation.

---

## 2. Case Study 1: Design a Parking Lot

### 2.1 Requirements
- Multi-floor parking lot with distinct spot types: Motorcycle, Compact (Car), and Large (Truck).
- Automatically assigns the nearest available spot of the appropriate type.
- Generates a timestamped ticket on entry and calculates hourly fees on exit.

### 2.2 Complete Python Implementation
```python
from enum import Enum
from abc import ABC, abstractmethod
import time

class VehicleType(Enum):
    MOTORCYCLE = 1
    CAR = 2
    TRUCK = 3

class SpotType(Enum):
    MOTORCYCLE = 1
    COMPACT = 2
    LARGE = 3

class Vehicle(ABC):
    def __init__(self, license_plate: str, vehicle_type: VehicleType):
        self.license_plate = license_plate
        self.vehicle_type = vehicle_type

class Car(Vehicle):
    def __init__(self, license_plate: str):
        super().__init__(license_plate, VehicleType.CAR)

class Motorcycle(Vehicle):
    def __init__(self, license_plate: str):
        super().__init__(license_plate, VehicleType.MOTORCYCLE)

class ParkingSpot:
    def __init__(self, spot_id: int, spot_type: SpotType):
        self.spot_id = spot_id
        self.spot_type = spot_type
        self.parked_vehicle: Vehicle | None = None

    @property
    def is_available(self) -> bool:
        return self.parked_vehicle is None

    def can_fit_vehicle(self, vehicle: Vehicle) -> bool:
        if vehicle.vehicle_type == VehicleType.MOTORCYCLE:
            return True
        if vehicle.vehicle_type == VehicleType.CAR:
            return self.spot_type in (SpotType.COMPACT, SpotType.LARGE)
        if vehicle.vehicle_type == VehicleType.TRUCK:
            return self.spot_type == SpotType.LARGE
        return False

    def park(self, vehicle: Vehicle) -> bool:
        if self.is_available and self.can_fit_vehicle(vehicle):
            self.parked_vehicle = vehicle
            return True
        return False

    def vacate(self) -> None:
        self.parked_vehicle = None

class Ticket:
    def __init__(self, ticket_id: str, vehicle: Vehicle, spot: ParkingSpot):
        self.ticket_id = ticket_id
        self.vehicle = vehicle
        self.spot = spot
        self.entry_time = time.time()

class ParkingLot:
    """Singleton Controller managing parking operations."""
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.spots: list[ParkingSpot] = []
            cls._instance.active_tickets: dict[str, Ticket] = {}
        return cls._instance

    def add_spot(self, spot: ParkingSpot):
        self.spots.append(spot)

    def park_vehicle(self, vehicle: Vehicle) -> Ticket | None:
        for spot in self.spots:
            if spot.is_available and spot.can_fit_vehicle(vehicle):
                spot.park(vehicle)
                ticket_id = f"TICK-{vehicle.license_plate}-{int(time.time())}"
                ticket = Ticket(ticket_id, vehicle, spot)
                self.active_tickets[ticket_id] = ticket
                print(f"Vehicle {vehicle.license_plate} parked at Spot {spot.spot_id}")
                return ticket
        print(f"No available spots for {vehicle.license_plate}!")
        return None

    def unpark_vehicle(self, ticket_id: str) -> float:
        ticket = self.active_tickets.pop(ticket_id, None)
        if not ticket:
            raise ValueError("Invalid ticket ID")
        ticket.spot.vacate()
        duration_hours = max(1.0, (time.time() - ticket.entry_time) / 3600)
        fee = duration_hours * 20.0  # $20 per hour rate
        print(f"Vehicle {ticket.vehicle.license_plate} exited. Total Fee: ${fee:.2f}")
        return fee
```

---

## 3. Case Study 2: Design an LRU Cache (O(1) Get & Put)

The Least Recently Used (LRU) Cache discards the least recently accessed items first when reaching capacity. It requires:
- $O(1)$ `get(key)`
- $O(1)$ `put(key, value)`

### Architecture: Doubly Linked List + Hash Map
- **Hash Map**: Maps `key -> Node` for $O(1)$ lookup.
- **Doubly Linked List**: Maintains chronological order of usage. The most recently used node is kept at the `head`, and the least recently used node is at the `tail`.

```python
class DListNode:
    def __init__(self, key: int = 0, val: int = 0):
        self.key = key
        self.val = val
        self.prev: DListNode | None = None
        self.next: DListNode | None = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache: dict[int, DListNode] = {}
        # Pseudo-nodes to eliminate boundary edge checks
        self.head = DListNode()
        self.tail = DListNode()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: DListNode) -> None:
        """Splices node out of doubly linked list."""
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_head(self, node: DListNode) -> None:
        """Inserts node immediately after dummy head."""
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._add_to_head(node)  # Mark as most recently used
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            node.val = value
            self._remove(node)
            self._add_to_head(node)
        else:
            if len(self.cache) >= self.capacity:
                # Evict least recently used (node before dummy tail)
                lru_node = self.tail.prev
                self._remove(lru_node)
                del self.cache[lru_node.key]
            
            new_node = DListNode(key, value)
            self.cache[key] = new_node
            self._add_to_head(new_node)
```

---

## 4. Case Study 3: Design an Elevator System

### Core Entities & Scheduling Algorithm
- **ElevatorCar**: Maintains current floor, direction (`UP`, `DOWN`, `IDLE`), door status, and an internal destination set.
- **Direction**: `Enum(UP, DOWN, IDLE)`.
- **LOOK / SCAN Algorithm**: The elevator continues traveling in its current direction servicing all requests ahead of it until there are no further requests in that direction, then reverses direction.

```python
class Direction(Enum):
    UP = 1
    DOWN = 2
    IDLE = 3

class ElevatorCar:
    def __init__(self, car_id: int):
        self.car_id = car_id
        self.current_floor = 0
        self.direction = Direction.IDLE
        self.destinations: set[int] = set()

    def add_destination(self, floor: int):
        self.destinations.add(floor)
        if self.direction == Direction.IDLE:
            if floor > self.current_floor:
                self.direction = Direction.UP
            elif floor < self.current_floor:
                self.direction = Direction.DOWN

    def step(self):
        """Simulates elevator moving one floor."""
        if self.direction == Direction.UP:
            self.current_floor += 1
            if self.current_floor in self.destinations:
                self.destinations.remove(self.current_floor)
                print(f"[Elevator {self.car_id}] Arrived at Floor {self.current_floor}.")
            if not any(f > self.current_floor for f in self.destinations):
                self.direction = Direction.DOWN if self.destinations else Direction.IDLE

        elif self.direction == Direction.DOWN:
            self.current_floor -= 1
            if self.current_floor in self.destinations:
                self.destinations.remove(self.current_floor)
                print(f"[Elevator {self.car_id}] Arrived at Floor {self.current_floor}.")
            if not any(f < self.current_floor for f in self.destinations):
                self.direction = Direction.UP if self.destinations else Direction.IDLE
```
