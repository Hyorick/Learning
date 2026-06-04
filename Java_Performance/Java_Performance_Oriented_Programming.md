# Java Performance Oriented Programming / Collections

**Table of Content**
- Stack vs Heap Memory
  - Stack Memory ✅
  - Heap Memory ✅
  - Summary ✅
- Programmation Dynamique (2 approaches)
  - **Memoization**	: `Top-down` **`recursion`** + **`cache`** ✅
  - **Tabulation**	: `Bottom-up` **`iterative`**. (loops) ✅
- Collections Complexity
  - HashSet/Hashmap put/add behaviour (important) ✅
  - How HashMap checks key uniqueness ✅
- Concurrency And Multi-Threading
  - Big Common Problem in Multi-thread programming
  - Thread Creation 
    - Thread lifecycle
    - **Extends** `Thread` ✅
    - ***Implements*** `Runnable` (more common) ✅
    - 👉 ***Thread pool*** : The `ExecutorService` Interface (**Best**) 📋
        - Thread management in thread pool 📋
        - Scheduling Tasks 📋
        - Executors Factory Methods 📋
  - Threading Problems
    - Deadlock
    - Starvation
    - Livelock
    - Race conditions
    - 🚨Contention : **Too many threads compete** for same lock/resource.
    - Visibility Problems (Solution : `volatile`)
    - Missed Signal / **Lost Notification** : Thread sends `notify()` before another thread starts waiting
    - 🚨Thread **Leakage**: Threads created but **never stopped**. (Solution : `Thread pools`, `ExecutorService` shutdown)
  - Thread-Safety Mechanisms
    - Synchronized on static vs instance method, synchronized block to protect static vs instance attribute ✅
    - Atomic classes ✅
  - Concurrent & Synchronized Collections
    - Concurrent Collections ✅
    - Synchronized Collections ✅
    - Primitives Collections ✅
  

## Stack vs Heap Memory

### Stack Memory

`Stack Memory` in Java is **used** for **static memory** **allocation** and the **execution** of a **thread**

It contains 

* >**`primitive values`** that are specific to a method 
* >and **`references`** to **objects** referred **from the method** that are in a heap.
* >Store also **`method calls`**
* Access to this memory is in `Last-In-First-Out (LIFO)` **order (Pile)**
* >Whenever we **call a new method** (Every method call creates a **stack frame**), a **new block** is **created on top** of the **stack** which contains **values specific to that method**, 

  * like **primitive variables** 
  * and **references** to **object**
* When the **method finishes execution,** its corresponding **stack frame is flushed,** the flow goes back to the calling method, **and space becomes available for the next method**

Key Features

* 💡It **grows and shrinks** as **new methods** are **called and returned,** respectively.
* **💡Variables** **inside** the stack **exist only as long as the method** that created them **is running**.
* It’s **automatically allocated and deallocated** when the **method finishes execution**.
* If this **memory is full**, Java throws **java.lang.StackOverFlowError**.
* **Access** to this memory is **fast** when **compared** **to** ***heap memory***.
* 💡This memory is **threadsafe**, as **each thread** operates in **its own stack**.


### Heap Memory
`Heap space` is **used for** the **dynamic memory allocation** of **Java objects** and **JRE classes** at **runtime**.

>**JRE: J**ava **Runtime E**nvironment, is software that **Java programs require to run correctly**

* >**New objects** are always **created** in ***heap space***, 
* >and the **references** to **these objects** are stored in ***stack memory*** (of **current thread**)
* These **objects have global access** and we can **access them from anywhere** **in the application**.
* >Store also `Arrays`and `instance variables`

We can break this memory model down into smaller parts, called **generations**, which are:

* >💡**Young Generation** – this is **where** all **new objects** are **allocated and aged**. A minor `Garbage collection` occurs when this fills up.
* >💡**Old or Tenured Generation** – this is **where** **long surviving objects** are **stored**. When objects are stored in the ***Young Generation***, a **threshold for the object’s age is set**, and when that **threshold is reached**, the **object is moved** to the ***old generation***.
* **Permanent Generation** – this consists of **JVM metadata** for the `runtime` **classes** and application **methods**.

We can always manipulate the size of heap memory as per our requirement. For more information, visit this linked Baeldung article.

Key Features

* It’s **accessed** **via** **complex memory management** techniques that include the **Young Generation**, **Old or Tenured Generation**, and **Permanent Generation**.
* If `heap space` **is full**, Java throws **java.lang.OutOfMemoryError.**
* **Access** to this memory is **comparatively slower** **than** ***stack memory***
* ⚠️ This memory, in contrast to ***stack***, **isn’t automatically deallocated**. 📢 It **needs** `Garbage Collector`** 🚛 to **free up unused objects** so as to keep the efficiency of the memory usage.
* Unlike stack, 🚨a heap **isn’t threadsafe** and 📢 needs to **be guarded by properly *synchronizing*** the code. (`Programmation Multi-Thread`)

### Summary

| | Stack Memory | Heap Space |
|---|---|---|
| Application | `Stack` is used in parts, one at a time during execution of a **thread** | The entire application uses `Heap` space during **runtime** |
| Size | `Stack` has **size limits depending upon OS**, and is usually **smaller than** `Heap` | There is **no size limit** on `Heap` |
| Storage | Stores only **primitive variables** and **references to objects** that are created in `Heap` Space | All the **newly created objects** are stored here |
| Order (Methode d'accès au données) | It’s accessed using **Last-in First-out (LIFO)** memory allocation system | This memory is accessed via 🚩complex memory management techniques that include **Young Generation**, **Old or Tenured Generation**, and **Permanent Generation**. |
| Life | `Stack` memory only **exists as long as the current method** is **running** | `Heap` space **exists as long as** the **application runs** |
| Efficiency (Allocation/Access) | **Much faster**🚀 to allocate when compared to `heap` | **Slower**🐢 to allocate when compared to `stack` |
| Allocation/Deallocation | This Memory is **automatically allocated** and **deallocated** when a **method** is **called** and **returned**, respectively | `Heap` space is **allocated** when **new objects** are **created** and **deallocated** by `Garbage Collector` 🚛 when they’re no longer referenced |


## Programmation Dynamique
- **Breaking** a *problem* **into smaller subproblems**
- **Saving** already **computed results**
- **Reusing them** instead of recalculating

<pre> The main goal is: Avoid repeated work</pre>

### Solution

Dynamic programming stores previous results.

Two main approaches:
>- **Memoization**	: `Top-down` **`recursion`** + **`cache`**
>- **Tabulation**	: `Bottom-up` **`iterative`**.
Instead of recursion, build results step by step.

### Exemple
Fibonacci : fib(n) = fib(n-1) + fib(n-2)

> **Normal Recursive** Fibonacci
<pre>
public static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    return fib(n - 1) + fib(n - 2);
}
</pre>

>**Memoization** (Top-Down : du Haut n vers le bas 0)

We save already computed values.
<pre>
static Map«Integer, Integer» cache = new HashMap<>();

public static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    if (cache.containsKey(n)) {
        return cache.get(n);
    }

    int result = fib(n - 1) + fib(n - 2);

    cache.put(n, result);

    return result;
}
</pre>
For fib(5) we have
- First time: fib(4) computed
- Next time: fib(4) retrieved from cache. No recomputation

> **Tabulation** (Bottom-up : du bas 0 vers le haut n)

Instead of recursion, build results step by step
<pre>
public static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    int[] dp = new int[n + 1];

    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[n];
}
</pre>

>**Optimized Iterative**

Uses only 2 variables instead of array.
<pre>
public static int fib(int n) {

    if (n <= 1) {
        return n;
    }

    int prev2 = 0;
    int prev1 = 1;

    for (int i = 2; i <= n; i++) {

        int current = prev1 + prev2;

        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
</pre>

| Version | Time | Space |
|---|---|---|
| Recursive factorial | O(n) | O(n) |
| Iterative factorial | O(n) | O(1) |

| Version | Time | Space |
|---|---|---|
| Recursive fibonacci | O(2^n) | O(n) |
| Recursive fibonacci (Memoization) | O(n) | O(n) |
| fibonacci (Tabulation) | O(n) | O(n) |
| Optimized Iterative Fibonacci | O(n) | O(1) |

## Collections Complexity
| Type  | Implementation  | Internal Structure       | Add / Put                                                                        | Get / Contains                                            | Remove                                                    | Ordering               |
| ----- | --------------- | ------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------- | ---------------------- |
| List  | `ArrayList`     | Dynamic Array            | Avg: O(1), Worst: O(n)                                                           | Avg: O(1), Worst: O(1)                                    | Avg: O(n), Worst: O(n)                                    | Insertion order        |
| List  | `LinkedList`    | Doubly Linked List       | Avg: O(1), Worst: O(1)                                                           | Avg: O(n), Worst: O(n)                                    | Avg: O(1), Worst: O(1)                                    | Insertion order        |
| List  | `Vector`        | Dynamic Array            | Avg: O(1), Worst: O(n)                                                           | Avg: O(1), Worst: O(1)                                    | Avg: O(n), Worst: O(n)                                    | Insertion order        |
| Set   | `HashSet`       | Hash Table (HashMap)     | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify)                        | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | No order               |
| Set   | `LinkedHashSet` | Hash Table + Linked List | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify)                        | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Insertion order        |
| Set   | `TreeSet`       | Red-Black Tree           | Avg: O(log n), Worst: O(log n)                                                   | Avg: O(log n), Worst: O(log n)                            | Avg: O(log n), Worst: O(log n)                            | Sorted                 |
| Map   | `HashMap`       | Hash Table               | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify)                        | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | No order               |
| Map   | `LinkedHashMap` | Hash Table + Linked List | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify)                        | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify) | Insertion/access order |
| Map   | `TreeMap`       | Red-Black Tree           | Avg: O(log n), Worst: O(log n)                                                   | Avg: O(log n), Worst: O(log n)                            | Avg: O(log n), Worst: O(log n)                            | Sorted by key          |
| Map   | `Hashtable`     | Hash Table               | Avg: O(1), Worst: O(n) (or O(log n) after Java 8 treeify concept does NOT apply) | Avg: O(1), Worst: O(n)                                    | Avg: O(1), Worst: O(n)                                    | No order               |
| Queue | `PriorityQueue` | Binary Heap              | Avg: O(log n), Worst: O(log n)                                                   | Peek: O(1), Worst: O(1)                                   | Avg: O(log n), Worst: O(log n)                            | Priority order         |
| Queue | `ArrayDeque`    | Circular Array           | Avg: O(1), Worst: O(n) (resize)                                                  | Avg: O(1), Worst: O(1)                                    | Avg: O(1), Worst: O(n) (resize)                           | FIFO/LIFO              |


| Type  | Implementation  | Notes                    |
| ----- | --------------- | ------------------------ |
| List  | `ArrayList`     | Fast random access       |
| List  | `LinkedList`    | Good for insert/remove   |
| List  | `Vector`        | Synchronized             |
| Set   | `HashSet`       | Backed by `HashMap`      |
| Set   | `LinkedHashSet` | Hash table + linked list |
| Set   | `TreeSet`       | Red-Black Tree           |
| Map   | `HashMap`       | Most used map            |
| Map   | `LinkedHashMap` | Keeps ordering           |
| Map   | `TreeMap`       | Red-Black Tree           |
| Map   | `Hashtable`     | Legacy synchronized      |
| Queue | `PriorityQueue` | Heap-based               |
| Queue | `ArrayDeque`    | Faster than Stack        |

>`HashMap` / `HashSet` **Worst Case**

Worst case happens when:

All keys land in same bucket due to bad hashing

Then:
- **Before** Java 8: **O(n)** (linked list traversal)
- **Java 8+**: **O(log n)** (**treeified** bucket using Red-Black Tree)

Pour les HashMap lorsqu'on à 2 clés ayant le même Hashcode, les valeurs pour ses clés sont stockés dans un bucket qui se comporte:
- **Before** Java 8: en linked list
- **Java 8+**: en arbre équilibré 

### How HashMap checks key uniqueness
When you do:

map.put(key, value);

Java internally does this:

- >**Step 1 — hashCode**()

key.hashCode() → determines bucket index

Example:

"John" → hashCode = 12345 → bucket 5

So first it decides:

👉 “Which bucket should I look in?”

- >**Step 2 — equals()**

Inside that bucket, Java checks:

existingKey.equals(newKey)

This is used to decide:

👉 “Is this the same key or a new one?”
<pre>
Full Flow (Very Important)
put(key, value)
   ↓
1. hashCode() → find bucket
   ↓
2. if bucket empty → insert
   ↓
3. if bucket not empty:
       ↓
   compare equals()
       ↓
   if equals = true → replace value
   if equals = false → add new entry
</pre>

## Concurrency And Multi-Threading

Synchronizers: coordinate the actions of multiple threads,

Locks:

Scheduling Tasks
### Thread Creation
- Thread LifeCycle
<pre>
                                   ┌─────────┐
                                   │   NEW   │
                                   │ Thread  │
                                   │ created │
                                   └────┬────┘
                                        │
                                        │ start()
                                        │
                                        ▼
                         CPU scheduled / eligible to run
                                ┌──────────────┐
                ┌──────────────▶│   RUNNABLE   │◀──────────────────────┐
                │               │ running OR   │                       │
                │               │ ready to run │                       │
                │               └──────┬───────┘                       │
                │                      │                               │
                │                      │ run() completes               │
                │                      │ uncaught exception            │
                │                      ▼                               │
                │               ┌─────────────┐                        │
                │               │ TERMINATED  │                        │
                │               │ thread dead │                        │
                │               └─────────────┘                        │
                │                                                      │
                │                                                      │
                │ trying to enter synchronized block/method            │
                │ while another thread owns monitor lock               │
                ▼                                                      │
         ┌──────────────┐                                               │
         │   BLOCKED    │───────────────────────────────────────────────┘
         │ waiting for  │
         │ monitor lock │
         └──────┬───────┘
                │
                │ monitor lock acquired
                │
                ▼
         ┌──────────────┐
         │   RUNNABLE   │
         └──────────────┘


                wait()
                join()
                LockSupport.park()
                Condition.await()
                         │
                         ▼
                ┌──────────────┐
                │   WAITING    │
                │ waiting      │
                │ indefinitely │
                └──────┬───────┘
                       │
                       │ notify()
                       │ notifyAll()
                       │ unpark()
                       │ joined thread ends
                       │ Condition.signal()
                       │ interrupt()
                       ▼
                ┌──────────────┐
                │   BLOCKED    │
                │ may need     │
                │ monitor lock │
                └──────┬───────┘
                       │
                       │ lock reacquired
                       ▼
                ┌──────────────┐
                │   RUNNABLE   │
                └──────────────┘



                sleep(time)
                wait(time)
                join(time)
                parkNanos()
                parkUntil()
                Condition.await(time)
                         │
                         ▼
                ┌────────────────┐
                │ TIMED_WAITING  │
                │ waiting with   │
                │ timeout        │
                └───────┬────────┘
                        │
                        │ timeout expires
                        │ OR notify()
                        │ OR notifyAll()
                        │ OR interrupt()
                        │ OR unpark()
                        ▼
                ┌──────────────┐
                │   BLOCKED    │
                │ may need     │
                │ monitor lock │
                └──────┬───────┘
                       │
                       │ lock reacquired
                       ▼
                ┌──────────────┐
                │   RUNNABLE   │
                └──────────────┘
</pre>

**wait()** :
- thread **releases lock**
- thread enters `WAITING` state
- Utilisé dans un **block synchronized**
- utilisé avec `notify()` (wakes ONE waiting thread) ou `notifyAll()` (Utilisé dans un **block synchronized**)

- **Extends** `Thread`
<pre>
public class MyThread extends Thread {
    public void run() {
        System.out.println("New thread is running");
    } 
}

MyThread myThread = new MyThread();
// launch the new thread
myThread.start();
</pre>

- ***Implements*** `Runnable` (more common)
<pre>
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
//--------
🟧Option 1
public class MyRunnable implements Runnable {
    public void run() {
        System.out.println("New thread is running");
    }
} 

MyRunnable myRunnable = new MyRunnable();
Thread myThread = new Thread(myRunnable);
myThread.start();

//--------
or 🟧Option 2
Thread myThread = new Thread(() -> System.out.println("New thread is running")); //lambda expression
myThread.start();

</pre>

- Thread Pool with Executor, Executor Service Interface and Callable interface (**Méthode à privilégier**)

### Threading Problems

#### Deadlock
**Def**: Two or more threads wait forever for each other’s locks.

💡To avoid deadlocks, it's important to follow best practices such as:

- **Acquiring locks in a consistent order**: If multiple locks need to be acquired, they should be acquired in the same order by all threads to avoid circular wait conditions.

- **Timeout mechanisms**: Use timeout mechanisms when attempting to acquire locks, so that threads don't wait indefinitely if they are unable to acquire a lock.

- **Resource ordering**: Assign a numerical order to resources and ensure that threads acquire resources in ascending order to prevent circular wait conditions.

- **Lock granularity**: Use fine-grained locks when possible, locking only the necessary sections of code to reduce the likelihood of contention and deadlocks.

- Reduce **nested locking**

#### Starvation
**Def**:
- A thread never gets CPU time or lock access because others monopolize resources.
- High-priority thread always runs, Low-priority thread rarely executes or unfair locking.

💡To mitigate starvation, consider the following approaches:

- **Fair scheduling**: Use fair scheduling mechanisms, such as `fair locks` or `semaphores`, which ensure that threads are granted access to shared resources in the order they requested them.

- **Avoid long-running tasks**: Break down long-running tasks into smaller units of work, allowing other threads to have a chance to execute in between.

- **Adjust thread priorities**: Assign appropriate priorities to threads based on their importance and resource requirements. However, be careful when manipulating thread priorities, as it can lead to complex and hard-to-predict behavior.

- **Timeout mechanisms**: Implement timeout mechanisms that allow threads to abandon waiting for a resource if they have been waiting for too long.

#### Livelock
**Def**: Threads keep reacting to each other but never make progress. Unlike ***deadlock***, threads are active but uselessly

💡To resolve livelocks, consider the following approaches:

- **Randomized backoff**: Introduce randomness in the yielding mechanism. Instead of immediately yielding, threads can wait for a random amount of time before retrying. This reduces the likelihood of threads continuously yielding to each other in a synchronized manner.

- **Resource ordering**: Assign a specific order to the resources or conditions that threads are waiting for. Ensure that threads acquire resources or check conditions in a consistent order to avoid circular dependencies.

- **Timeout mechanisms**: Implement timeout mechanisms that allow threads to abandon waiting and take alternative actions if they have been waiting for too long. This prevents threads from indefinitely yielding to each other.

- **Locking strategies**: Use appropriate locking strategies, such as **read-write locks** or **fine-grained locks**, to minimize contention and reduce the chances of ***livelock***.

#### Race Conditions
**Def**: Multiple threads access and modify shared data simultaneously, producing unpredictable results.

💡To prevent race conditions, it's essential to use synchronization mechanisms that ensure exclusive access to shared resources. Some common techniques include:

- **Locks**: Use lock objects, such as `ReentrantLock` (from `java.util.concurrent.locks`) or `synchronized blocks`, to ensure that only one thread can access the shared resource at a time.

- **Atomic variables**: Use atomic variables, such as `AtomicInteger`, which provide thread-safe operations for reading and writing shared variables.

- **Concurrent data structures**: Utilize thread-safe data structures from the `java.util.concurrent` package, such as `ConcurrentHashMap` or `CopyOnWriteArrayList`, which are designed to handle concurrent access.

- **Synchronization primitives**: Employ synchronization primitives like `semaphores`, `barriers`, or `latches` to coordinate thread execution and access to shared resources.

### Thread-Safety Mechanisms
<pre>
┌───────────────────────────────────────────────────────────┐
│             Thread-Safety Mechanisms                      │
│                                                           │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  volatile   │    │   Atomic     │    │ synchronized │  │
│  │             │    │   Classes    │    │              │  │
│  │ Visibility  │    │ Atomicity of │    │  Exclusivity │  │
│  │ guarantee   │    │  operations  │    │ of execution │  │
│  └─────────────┘    └──────────────┘    └──────────────┘  │
│                                                           │
│  ┌─────────────┐    ┌───────────────┐    ┌─────────────┐  │
│  │    Lock     │    │   Cyclic      │    │  Concurrent │  │
│  │  Interface  │    │   Barrier     │    │ Collections │  │
│  │             │    │               │    │             │  │
│  │ Fine-grained│    │Synchronization│    │ Thread-safe │  │
│  │   control   │    │     point     │    │ data struct │  │
│  └─────────────┘    └───────────────┘    └─────────────┘  │
│                                                           │
└───────────────────────────────────────────────────────────┘
</pre>

#### Synchronized on static vs instance method, synchronized block to protect static vs instance attribute

| Feature                                     | Instance Synchronization           | Static Synchronization                                 |
| ------------------------------------------- | ---------------------------------- | ------------------------------------------------------ |
| Purpose                                     | Protect instance (non-static) data | Protect static/shared class data                       |
| Lock owner                                  | Specific object instance           | Class object                                           |
| Lock used                                   | `this` or instance lock object     | `ClassName.class` or static lock                       |
| Scope                                       | Per object                         | Entire class                                           |
| Shared between instances?                   | No                                 | Yes                                                    |
| Typical protected data                      | `balance`, `name`, `count`         | `static cache`, `static counter`                       |
| Syntax (method)                             | `public synchronized void m()`     | `public static synchronized void m()`                  |
| Equivalent block form                       | `synchronized(this)`               | `synchronized(ClassName.class)`                        |
| Can two instances execute simultaneously?   | Yes                                | No (same class lock)                                   |
| Blocks other instance synchronized methods? | Only on SAME object                | Yes, for all static synchronized methods of that class |
| Blocks non-synchronized methods?            | No                                 | No                                                     |
| Best use case                               | Object state                       | Global shared state                                    |

| Method Type                  | Lock Used         | Blocks What?                                       |
| ---------------------------- | ----------------- | -------------------------------------------------- |
| synchronized instance method | `this`            | Other synchronized instance methods on SAME object |
| static synchronized method   | `ClassName.class` | Other static synchronized methods                  |
| non-synchronized method      | none              | Nothing                                            |
|👉 synchronized(**customLock**)     | custom object     | **Only code** using **same custom lock**                   |

---
>🚨💡Préférer 

`synchronized(**customLock**)`

<pre>
    // STATIC ATTRIBUTE
    private static int totalAccounts = 0;
    private static final Object STATIC_LOCK = new Object(); // protect static attribute "totalAccounts"
    
    // INSTANCE ATTRIBUTES
    private int balance;
    private String ownerName; // does not need protection
    private final Object balance_lock = new Object(); // protect instance attribute "balance"

    // Protect only balance
    public void deposit(int amount) {

        synchronized (BALANCE_LOCK) {
            balance += amount;
        }
    }
    // Protect only balance
    public void withdraw(int amount) {

        synchronized (BALANCE_LOCK) {

            if (balance >= amount) {
                balance -= amount;
            }
        }
    }

    // Reading ownerName does not need locking
    public String getOwnerName() {
        return ownerName;
    }

    // Static shared data
    public static int getTotalAccounts() {

        synchronized (STATIC_LOCK) {
            return totalAccounts;
        }
    }
</pre>
ou 
`ReentrantLock` (**Best**) give gives extra features:
- tryLock(),
- Timed waiting,
- Fair locking
- Interruptible locking
- Multiple condition variables Much more powerful than `wait()/notify()`.

au `Synchronized(this ou ClassName.class)`

#### Atomic classes
For **thread-safe operations** on **single variables**.
| Primitive Type | Atomic Class |
|---|---|
| `int` | `AtomicInteger` |
| `long` | `AtomicLong` |
| `boolean` | `AtomicBoolean` |
| `int[]` | `AtomicIntegerArray` |
| `long[]` | `AtomicLongArray` |
| `double` (accumulator style) | `DoubleAdder` |
| `long` (accumulator style) | `LongAdder` |
| reference type | `AtomicReference<V>` |
| reference array | `AtomicReferenceArray<V>` |

>`AtomicInteger/AtomicLong` only **one lock** : 
- Fast for Reading
- Writing: Ok for low contention (few threads/few concurrency)
- Writing: Worse for high contention (lots of threads/high concurrency)

>`LongAdder` (each thread write in copy variable so they are not blocked): 
- faster for writing
- Slower for Reading 

### Concurrent & Synchronized Collections

#### Concurrent Collections

When working with Java collections like `ArrayList`, `HashMap`, etc., in a **multi-threaded** environment, you may have encountered a `ConcurrentModificationException`. This exception is thrown when one thread is iterating over a collection while another thread tries to modify it structurally, for example, by adding or removing elements.

The **solution** is to use thread-safe, **concurrent collections** instead. Java provides several concurrent collection classes that allow **multiple threads to access and modify them safely**, without the risk of `ConcurrentModificationException`.

>!! Most classic synchronized collections (`Vector`, `Hashtable`, `synchronizedXXX`) in Java use a **single lock**for the **entire collection**. This is concurrent version (`ConcurrentHashMap` etc) **better to use** for **high concurrency**, they have **multiple locks** for the collection.

| Type | Implementation | Concurrent Version |
|---|---|---|
| List | `ArrayList` | `CopyOnWriteArrayList<E>` |
| Set | `HashSet` | `ConcurrentHashMap.newKeySet()` |
| Set | `LinkedHashSet` | `CopyOnWriteArraySet<E>` |
| Set | `TreeSet` | `ConcurrentSkipListSet<E>` |
| Map | `HashMap` | `ConcurrentHashMap<K,V>` |
| Map | `LinkedHashMap` | `ConcurrentHashMap<K,V>` |
| Map | `TreeMap` | `ConcurrentSkipListMap<K,V>` |
| Queue | `Queue` | `ConcurrentLinkedQueue<E>`: LinkedList aussi |
| Queue | `ArrayDeque` | `ConcurrentLinkedDeque<E>` |
| Queue | `PriorityQueue` | `PriorityBlockingQueue<E>` |
| Queue | `Deque` | `LinkedBlockingDeque<E>` |
| Queue | `BlockingQueue` | `LinkedBlockingQueue<E>`: LinkedList aussi |

#### Synchronized Collections
| Type | Implementation | Synchronized Version |
|---|---|---|
| List | `ArrayList` | `Vector` |
| List | `LinkedList` | Collections.`synchronizedList()` |
| Set | `HashSet` | Collections.`synchronizedSet()` |
| Set | `LinkedHashSet` | Collections.`synchronizedSet()` |
| Set | `TreeSet` | Collections.`synchronizedSortedSet()` |
| Set | `NavigableSet` | Collections.`synchronizedNavigableSet()` |
| Map | `HashMap` | `Hashtable` |
| Map | `LinkedHashMap` | Collections.`synchronizedMap()` |
| Map | `TreeMap` | Collections.`synchronizedSortedMap()` |
| Map | `NavigableMap` | Collections.`synchronizedNavigableMap()` |
| Queue | `PriorityQueue` | `PriorityBlockingQueue` |

In general, it's preferable to use the ***concurrent collection*** classes directly, as they are **designed from the ground up for high concurrency**. The ***synchronization wrappers*** are useful when you need to add thread-safety to an existing collection or when using a less common collection type that doesn't have a direct concurrent equivalent.

#### Primitives Collections
Most of these come from `fastutil`. Il y a aussi pour `byte`, `short`, `float`, `boolean` et `char`. 
- high-performance primitive storage
    - **no boxing**
    - no Integer objects
    - direct int[]-backed storage

| Type | Standard Collection | Primitive Collection |
|---|---|---|
| List | `List<Integer>` | `IntArrayList` |
| List | `List<Long>` | `LongArrayList` |
| List | `List<Double>` | `DoubleArrayList` |
| Set | `Set<Integer>` | `IntOpenHashSet` |
| Set | `Set<Long>` | `LongOpenHashSet` |
| Set | `Set<Double>` | `DoubleOpenHashSet` |
| Map | `Map<Integer,Integer>` | `Int2IntOpenHashMap` |
| Map | `Map<Long,Long>` | `Long2LongOpenHashMap` |
| Map | `Map<Integer,String>` | `Int2ObjectOpenHashMap<V>` |
| Queue | `Queue<Integer>` | `IntArrayFIFOQueue` |
| Queue | `PriorityQueue<Integer>` | `IntHeapPriorityQueue` |

| Type | Implementation | Synchronized Version |
|---|---|---|
| String | `StringBuilder` (faster for single-threaded  code) | `StringBuffer`(multiple threads modify the same string object) |

### Scheduling Tasks

