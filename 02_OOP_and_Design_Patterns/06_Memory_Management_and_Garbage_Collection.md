# Module 02: OOP — Memory Management & Garbage Collection

---

## 1. Stack vs. Heap Allocation

Understanding how memory is partitioned during program execution is essential for diagnosing performance bottlenecks, garbage collection pauses, and memory leaks.

```
┌───────────────────────────────────────┐
│              PROCESS RAM              │
├───────────────────────────────────────┤
│ STACK MEMORY                          │
│ • Local primitive variables           │
│ • Function call frames & return addrs │
│ • Fast LIFO allocation / deallocation │
│ • Fixed size (overflow = StackOverflow│
├───────────────────────────────────────┤
│ HEAP MEMORY                           │
│ • Dynamic object instances            │
│ • Long-lived data structures          │
│ • Managed by Garbage Collector        │
│ • Slower allocation (fragmentation)   │
└───────────────────────────────────────┘
```

---

## 2. Python Memory Management

Python uses a dual-tier memory management system: **Reference Counting** supplemented by a **Generational Cyclic Garbage Collector**.

### 2.1 Reference Counting (Primary Mechanism)
Every Python object (`PyObject`) maintains an internal metadata field: `ob_refcnt`.
- **Incremented** when: assigned to a variable (`b = a`), passed into a function, or added to a list/dict.
- **Decremented** when: variable goes out of scope, reassigned, or explicitly deleted via `del a`.
- **Deallocation**: The moment `ob_refcnt == 0`, the memory occupied by the object is immediately reclaimed.

```python
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # Typically 2 (x + argument to getrefcount)
y = x
print(sys.getrefcount(x))  # Incremented to 3
del y
print(sys.getrefcount(x))  # Decremented back to 2
```

### 2.2 The Cyclic Garbage Collector (Generational GC)
Reference counting alone fails when objects point to each other in a circular loop, because their reference count never drops to zero:

```python
class Node:
    def __init__(self):
        self.partner = None

a = Node()
b = Node()
a.partner = b
b.partner = a

del a  # Reference count is still 1 (pointed to by b.partner)
del b  # Reference count is still 1 (pointed to by a.partner)
# Orphaned in memory! Pure reference counting would leak this forever.
```

#### How Python Resolves Cycles: 3 Generations
Python groups all dynamic container objects into three generations (Gen 0, Gen 1, Gen 2):
1. **Generation 0**: Newly created objects. Scanned most frequently.
2. **Generation 1**: Objects that survive a Gen 0 garbage collection pass.
3. **Generation 2**: Long-lived objects that survive Gen 1 collection passes. Scanned least frequently.

Python uses a **Mark-and-Sweep / Cycle Detection** algorithm on Gen 0: it isolates container objects, temporarily decrements reference counts to eliminate internal pointers, and flags objects with zero external references for deletion.

---

## 3. Java JVM Memory Management

The Java Virtual Machine (JVM) employs an advanced generational heap architecture:

```
┌────────────────────────────────────────────────────────────────────────┐
│                                JVM HEAP                                │
├────────────────────────────────────────┬───────────────────────────────┤
│            YOUNG GENERATION            │     OLD (TENURED) GENERATION  │
│ ┌──────────────┬───────┬───────┐       │                               │
│ │  Eden Space  │  S0   │  S1   │       │ Long-lived objects promoted   │
│ │ (New Objects)│ (From)│ (To)  │       │ after surviving multiple GCs  │
│ └──────────────┴───────┴───────┘       │                               │
└────────────────────────────────────────┴───────────────────────────────┘
```

### 3.1 The Generational Hypothesis
Most allocated objects have very short lifespans (e.g., local variables in a loop). Separating memory into Young and Old generations allows the garbage collector to focus on the Young generation without scanning the entire heap.

1. **Eden Space**: All newly instantiated objects (`new Object()`) begin here.
2. **Survivor Spaces (S0 & S1)**: When Eden fills, a **Minor GC** triggers. Surviving objects are moved to one survivor space, while dead objects are cleared. Surviving objects alternate between S0 and S1 with an incrementing "age" counter.
3. **Tenured (Old) Generation**: When an object survives a predetermined number of Minor GC cycles (e.g., 15 by default), it is promoted to the Tenured Generation.
4. **Major / Full GC**: Scans the entire heap (Young + Old). Causes a **Stop-The-World (STW)** pause where application threads halt temporarily.

---

## 4. Common Causes of Memory Leaks in OOP

Even in garbage-collected languages, memory leaks occur when references to unused objects are unintentionally retained.

### 4.1 Unbounded Caching / Global Collections
Appending objects to a global dictionary or list without eviction or size limits prevents the GC from reclaiming them.
- *Fix*: Use `collections.OrderedDict` with an eviction policy, or `weakref.WeakValueDictionary`.

### 4.2 Dangling Listeners / Observer Subscriptions
Subscribing an observer to a long-lived publisher without calling `unsubscribe()` keeps the observer in memory for the lifetime of the publisher.
- *Fix*: Use Weak References (`weakref.ref`) for observer callbacks.

```python
import weakref

class Publisher:
    def __init__(self):
        # Stores weak references: does NOT increment reference count
        self._subscribers = weakref.WeakSet()

    def subscribe(self, sub):
        self._subscribers.add(sub)
```
