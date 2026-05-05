Let's go deep on both. I'll start with **Java Concurrency** then move to **Order Book + FSM**.

------

## Part 1 — Java Concurrency Deep Dive

Here's a visual map of the full concurrency landscape before we go deep:Now let's go deep on each layer with code, interview traps, and trading context.

---

### Layer 1 — Primitives

**`volatile` — what it actually guarantees**

```java
// volatile guarantees: visibility + ordering. NOT atomicity.
private volatile boolean running = true;     // safe flag
private volatile int counter = 0;           // UNSAFE for counter++

// counter++ is 3 ops: read → increment → write
// Two threads can interleave → lost updates even with volatile
```

**Interview trap:** "Is `volatile` enough for a flag?" → Yes for a simple boolean stop signal. "Is `volatile int counter++` thread-safe?" → **No.** Use `AtomicInteger`.

**The Java Memory Model (JMM) — happens-before rules you must know:**

```java
// Rule 1: unlock HB lock
// Rule 2: volatile write HB volatile read
// Rule 3: thread.start() HB any action in thread
// Rule 4: thread action HB thread.join() returns
// Rule 5: transitive closure

volatile int x = 0;
int y = 0; // non-volatile

// Thread A:
y = 42;          // write to y
x = 1;           // volatile write to x (publishes y=42 too!)

// Thread B:
if (x == 1) {    // volatile read of x
    // guaranteed to see y == 42 due to HB chain
    System.out.println(y);
}
```

**`synchronized` internals — monitor & biased locking:**

```java
// Object monitor: each object has a monitor word (mark word in header)
// States: unlocked → biased → thin lock (CAS) → inflated (OS mutex)

public synchronized void method() { }       // lock on 'this'
synchronized (lock) { }                     // lock on specific object

// Double-checked locking — correct pattern (volatile required)
private volatile OrderBook instance;

public OrderBook getOrderBook() {
    if (instance == null) {                  // 1st check (no lock)
        synchronized (this) {
            if (instance == null) {          // 2nd check (with lock)
                instance = new OrderBook();  // volatile write publishes fully constructed object
            }
        }
    }
    return instance;
}
// Without volatile: partially constructed object can be visible to other threads
```

---

### Layer 2 — Explicit Locks

**`ReentrantLock` vs `synchronized` — when to choose:**

```java
ReentrantLock lock = new ReentrantLock(true); // fair=true: FIFO ordering

// tryLock — critical for trading (never block indefinitely)
if (lock.tryLock(100, TimeUnit.MICROSECONDS)) {
    try {
        // modify order book
    } finally {
        lock.unlock(); // ALWAYS in finally
    }
} else {
    // handle contention: queue the request, return NACK, circuit break
}

// lockInterruptibly — allows cancel on thread interrupt
lock.lockInterruptibly();
```

**`ReadWriteLock` for order book reads (high read, low write):**

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock  = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Many threads can read concurrently
public List<Order> getOrders(String symbol) {
    readLock.lock();
    try { return new ArrayList<>(orderMap.get(symbol)); }
    finally { readLock.unlock(); }
}

// Only one thread can write, blocks all readers
public void addOrder(Order order) {
    writeLock.lock();
    try { orderMap.computeIfAbsent(order.getSymbol(), k -> new ArrayList<>()).add(order); }
    finally { writeLock.unlock(); }
}
```

**`StampedLock` — optimistic reads (fastest for low-contention):**

```java
StampedLock sl = new StampedLock();

// Optimistic read — NO lock taken, just validate after
public double getBestBid() {
    long stamp = sl.tryOptimisticRead();
    double bid = this.bestBid;           // read WITHOUT lock
    if (!sl.validate(stamp)) {          // check if write happened
        stamp = sl.readLock();           // fallback to real read lock
        try { bid = this.bestBid; }
        finally { sl.unlockRead(stamp); }
    }
    return bid;
}
```

**Synchronizers — the ones you'll be asked to implement with:**

```java
// CountDownLatch — wait for N events (one-shot)
CountDownLatch venuesReady = new CountDownLatch(3); // 3 venues
// venue threads call: venuesReady.countDown();
venuesReady.await(500, TimeUnit.MILLISECONDS); // SOR waits max 500µs

// CyclicBarrier — synchronize N threads at a checkpoint (reusable)
CyclicBarrier barrier = new CyclicBarrier(4, () -> System.out.println("All filled"));
// each thread calls: barrier.await();

// Semaphore — limit concurrent access (e.g. venue throttle)
Semaphore throttle = new Semaphore(10); // 10 concurrent orders to venue
throttle.acquire();
try { sendToVenue(order); }
finally { throttle.release(); }
```

---

### Layer 3 — Executor Framework

**`ThreadPoolExecutor` — tune every parameter:**

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    4,                              // corePoolSize: always-alive threads
    8,                              // maximumPoolSize: max under load
    60, TimeUnit.SECONDS,           // keepAliveTime: idle thread TTL
    new ArrayBlockingQueue<>(1000), // bounded queue — critical for backpressure
    new ThreadFactory() {           // name threads for profiling
        AtomicInteger n = new AtomicInteger();
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "oms-worker-" + n.incrementAndGet());
            t.setDaemon(true);
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection: run in caller (backpressure)
    // AbortPolicy: throw (default), DiscardPolicy: silent drop, DiscardOldest: drop oldest
);
```

**`CompletableFuture` — async order flow:**

```java
CompletableFuture
    .supplyAsync(() -> riskCheck(order), riskExecutor)        // async risk
    .thenApplyAsync(passed -> route(passed), sorExecutor)     // async route
    .thenApplyAsync(venue -> sendFix(venue), fixExecutor)     // async send
    .exceptionally(ex -> {
        log.error("Order failed", ex);
        return cancelOrder(order);
    })
    .orTimeout(500, TimeUnit.MILLISECONDS)                    // SLA enforcement
    .thenAccept(result -> updateOrderState(result));
```

---

### Layer 4 — Lock-Free

**CAS (Compare-And-Swap) — the hardware instruction behind all atomics:**

```java
AtomicLong orderId = new AtomicLong(0);

// Under the hood: CAS(address, expected, newValue) — single CPU instruction
long next = orderId.incrementAndGet();  // atomic, no lock, no context switch

// Manual CAS loop — used in custom lock-free structures
AtomicReference<OrderState> state = new AtomicReference<>(OrderState.NEW);

public boolean transitionTo(OrderState expected, OrderState next) {
    return state.compareAndSet(expected, next); // returns false if someone else changed it
}

// CAS retry loop pattern
public void updateBestBid(double newBid) {
    double current;
    do {
        current = bestBid.get();
        if (newBid <= current) return; // stale update, abort
    } while (!bestBid.compareAndSet(current, newBid)); // retry if CAS fails
}
```

**`ConcurrentHashMap` internals (Java 8+):**

```java
// Java 8+: array of Node bins. Synchronized only on individual bins (not whole map)
// Read: volatile read of head node — NO lock
// Write: CAS on empty bins, synchronized on head for collisions
// TreeBin: bins with >8 nodes become red-black trees → O(log n) worst case

ConcurrentHashMap<String, Order> orders = new ConcurrentHashMap<>();

// Atomic compute — safe without external lock
orders.compute(orderId, (id, existing) -> {
    if (existing == null) return newOrder;
    return existing.withStatus(AMENDED);
});

// computeIfAbsent for price-level map in order book
priceLevel.computeIfAbsent(price, p -> new PriceLevel(p)).addOrder(order);
```

**False sharing — the silent killer in trading systems:**

```java
// Problem: two variables on same 64-byte cache line → thrashing between cores
class BadCounter {
    long counter1; // cache line: [counter1][counter2] → cores fight over same line
    long counter2;
}

// Fix with @Contended (JVM flag: -XX:-RestrictContended)
class GoodCounter {
    @Contended long counter1; // pads to own cache line
    @Contended long counter2;
}

// Manual padding (if @Contended not available)
class PaddedCounter {
    long p1, p2, p3, p4, p5, p6, p7; // 7 × 8 = 56 bytes padding
    volatile long counter;             // 8 bytes → total 64 bytes = 1 cache line
    long q1, q2, q3, q4, q5, q6, q7;
}
```

---

### Layer 5 — LMAX Disruptor (Trading Gold Standard)

```java
// Ring buffer: pre-allocated, fixed-size, no GC, sequence numbers not pointers
// Single Producer: one thread writes, multiple consumers read at own pace
// Multiple barriers for dependent processing stages

EventFactory<OrderEvent> factory = OrderEvent::new;
int bufferSize = 1024; // must be power of 2

Disruptor<OrderEvent> disruptor = new Disruptor<>(
    factory, bufferSize,
    DaemonThreadFactory.INSTANCE,
    ProducerType.SINGLE,           // lock-free for single producer
    new BusySpinWaitStrategy()     // lowest latency: CPU spins instead of sleeping
    // YieldingWaitStrategy: yields CPU (balance latency/throughput)
    // BlockingWaitStrategy: parks thread (lowest CPU use, highest latency)
);

// Chain handlers: risk check → route → persist (dependency graph)
disruptor.handleEventsWith(riskHandler)
         .then(routeHandler)
         .then(persistHandler);

RingBuffer<OrderEvent> ringBuffer = disruptor.start();

// Publish (zero allocation on hot path)
long sequence = ringBuffer.next();
try {
    OrderEvent event = ringBuffer.get(sequence);
    event.setOrder(order); // mutate pre-allocated object, no new()
} finally {
    ringBuffer.publish(sequence);
}
```

---

## Part 2 — Order Book + FSM (Code It Out)

Now the FSM state machine diagram, then full code:Now the full production-quality implementation:

---

### Order FSM — Thread-Safe Implementation

```java
// ─── Order State ──────────────────────────────────────────────────────────────
public enum OrderState {
    NEW, PENDING_ACK, ACKNOWLEDGED, PARTIALLY_FILLED, FILLED, CANCELLED, REJECTED;

    public boolean isTerminal() {
        return this == FILLED || this == CANCELLED || this == REJECTED;
    }
}

// ─── Valid Transitions Table ──────────────────────────────────────────────────
public enum OrderEvent {
    SUBMIT, ACK, REJECT, PARTIAL_FILL, FULL_FILL, CANCEL, AMEND
}
```

```java
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class OrderStateMachine {

    // Transition table: Map<CurrentState, Map<Event, NextState>>
    private static final Map<OrderState, Map<OrderEvent, OrderState>> TRANSITIONS;

    static {
        Map<OrderState, Map<OrderEvent, OrderState>> t = new EnumMap<>(OrderState.class);

        t.put(OrderState.NEW, Map.of(
            OrderEvent.SUBMIT,       OrderState.PENDING_ACK,
            OrderEvent.REJECT,       OrderState.REJECTED    // pre-trade risk fail
        ));
        t.put(OrderState.PENDING_ACK, Map.of(
            OrderEvent.ACK,          OrderState.ACKNOWLEDGED,
            OrderEvent.REJECT,       OrderState.REJECTED,
            OrderEvent.CANCEL,       OrderState.CANCELLED
        ));
        t.put(OrderState.ACKNOWLEDGED, Map.of(
            OrderEvent.PARTIAL_FILL, OrderState.PARTIALLY_FILLED,
            OrderEvent.FULL_FILL,    OrderState.FILLED,
            OrderEvent.CANCEL,       OrderState.CANCELLED,
            OrderEvent.AMEND,        OrderState.PENDING_ACK  // amend → re-ack cycle
        ));
        t.put(OrderState.PARTIALLY_FILLED, Map.of(
            OrderEvent.PARTIAL_FILL, OrderState.PARTIALLY_FILLED, // more partial fills
            OrderEvent.FULL_FILL,    OrderState.FILLED,
            OrderEvent.CANCEL,       OrderState.CANCELLED
        ));
        // Terminal states: no transitions out
        t.put(OrderState.FILLED,     Collections.emptyMap());
        t.put(OrderState.CANCELLED,  Collections.emptyMap());
        t.put(OrderState.REJECTED,   Collections.emptyMap());

        TRANSITIONS = Collections.unmodifiableMap(t);
    }

    // Thread-safe transition using CAS
    public OrderState transition(Order order, OrderEvent event) {
        OrderState current = order.getState();

        if (current.isTerminal()) {
            throw new IllegalStateException(
                "Order " + order.getId() + " is terminal: " + current);
        }

        Map<OrderEvent, OrderState> validEvents = TRANSITIONS.get(current);
        OrderState next = validEvents.get(event);

        if (next == null) {
            throw new IllegalStateException(
                "Invalid transition: " + current + " + " + event);
        }

        // CAS: only transition if state hasn't changed since we read it
        boolean success = order.compareAndSetState(current, next);
        if (!success) {
            // Another thread already transitioned — re-read and retry or fail
            throw new ConcurrentModificationException(
                "State changed concurrently for order " + order.getId()
                + " expected=" + current + " actual=" + order.getState());
        }

        order.addAuditEntry(current, event, next, System.nanoTime());
        return next;
    }
}
```

---

### Order — Thread-Safe with CAS State

```java
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.CopyOnWriteArrayList;
import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;

public class Order {

    public enum Side { BUY, SELL }
    public enum Type { MARKET, LIMIT, STOP, ICEBERG }

    // Immutable fields
    private final String id;
    private final String symbol;
    private final Side side;
    private final Type type;
    private final BigDecimal quantity;
    private final BigDecimal limitPrice; // null for market orders
    private final Instant createTime;

    // Mutable state — CAS-protected
    private final AtomicReference<OrderState> state;

    // Fill tracking — volatile for visibility
    private volatile BigDecimal filledQuantity = BigDecimal.ZERO;
    private volatile BigDecimal avgFillPrice   = BigDecimal.ZERO;

    // Audit trail — thread-safe append-only
    private final CopyOnWriteArrayList<AuditEntry> auditTrail = new CopyOnWriteArrayList<>();

    public Order(String id, String symbol, Side side, Type type,
                 BigDecimal quantity, BigDecimal limitPrice) {
        this.id          = id;
        this.symbol      = symbol;
        this.side        = side;
        this.type        = type;
        this.quantity    = quantity;
        this.limitPrice  = limitPrice;
        this.createTime  = Instant.now();
        this.state       = new AtomicReference<>(OrderState.NEW);
    }

    // CAS state transition
    public boolean compareAndSetState(OrderState expected, OrderState next) {
        return state.compareAndSet(expected, next);
    }

    // Apply a fill — synchronized to protect derived calculations
    public synchronized void applyFill(BigDecimal fillQty, BigDecimal fillPrice) {
        if (fillQty.compareTo(BigDecimal.ZERO) <= 0)
            throw new IllegalArgumentException("Fill qty must be positive");
        if (fillQty.compareTo(getRemainingQuantity()) > 0)
            throw new IllegalArgumentException("Fill exceeds remaining qty");

        // Update avg fill price: (prevAvg * prevFilled + fillPrice * fillQty) / newFilled
        BigDecimal newFilled = filledQuantity.add(fillQty);
        BigDecimal prevValue = avgFillPrice.multiply(filledQuantity);
        BigDecimal fillValue = fillPrice.multiply(fillQty);
        avgFillPrice   = prevValue.add(fillValue)
                                  .divide(newFilled, 8, java.math.RoundingMode.HALF_UP);
        filledQuantity = newFilled;
    }

    public BigDecimal getRemainingQuantity() {
        return quantity.subtract(filledQuantity);
    }

    public boolean isFullyFilled() {
        return filledQuantity.compareTo(quantity) >= 0;
    }

    void addAuditEntry(OrderState from, OrderEvent event, OrderState to, long nanoTs) {
        auditTrail.add(new AuditEntry(from, event, to, nanoTs));
    }

    public OrderState getState()          { return state.get(); }
    public String getId()                 { return id; }
    public String getSymbol()             { return symbol; }
    public BigDecimal getFilledQuantity() { return filledQuantity; }
    public List<AuditEntry> getAuditTrail() { return List.copyOf(auditTrail); }

    // Audit entry record
    public record AuditEntry(OrderState from, OrderEvent event,
                              OrderState to, long nanoTimestamp) {}
}
```

---

### Order Book — Thread-Safe with Price-Time Priority

```java
import java.math.BigDecimal;
import java.util.*;
import java.util.concurrent.ConcurrentSkipListMap;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.locks.StampedLock;

public class OrderBook {

    private final String symbol;

    // Bids: highest price first (descending) → TreeMap with reverse comparator
    // Asks: lowest price first (ascending)  → TreeMap with natural order
    // ConcurrentSkipListMap: lock-free concurrent sorted map
    private final ConcurrentSkipListMap<BigDecimal, PriceLevel> bids =
        new ConcurrentSkipListMap<>(Comparator.reverseOrder());

    private final ConcurrentSkipListMap<BigDecimal, PriceLevel> asks =
        new ConcurrentSkipListMap<>();

    // Index: orderId → Order for O(1) cancel/amend
    private final Map<String, Order> orderIndex = new ConcurrentHashMap<>();

    // Stamped lock for book-level reads (best bid/ask queries are very frequent)
    private final StampedLock sl = new StampedLock();

    public OrderBook(String symbol) {
        this.symbol = symbol;
    }

    public void addOrder(Order order, BigDecimal price) {
        ConcurrentSkipListMap<BigDecimal, PriceLevel> side =
            order.getSide() == Order.Side.BUY ? bids : asks;

        // computeIfAbsent is atomic in ConcurrentSkipListMap
        PriceLevel level = side.computeIfAbsent(price, PriceLevel::new);
        level.addOrder(order);
        orderIndex.put(order.getId(), order);
    }

    public boolean cancelOrder(String orderId) {
        Order order = orderIndex.remove(orderId);
        if (order == null) return false;

        ConcurrentSkipListMap<BigDecimal, PriceLevel> side =
            order.getSide() == Order.Side.BUY ? bids : asks;

        // Find price level (we'd store price in order in production)
        // Here simplified — production stores price on Order
        side.values().forEach(level -> level.removeOrder(orderId));
        return true;
    }

    // Optimistic read for best bid/ask (extremely hot path)
    public BigDecimal getBestBid() {
        long stamp = sl.tryOptimisticRead();
        Map.Entry<BigDecimal, PriceLevel> entry = bids.firstEntry();
        if (!sl.validate(stamp)) {
            stamp = sl.readLock();
            try { entry = bids.firstEntry(); }
            finally { sl.unlockRead(stamp); }
        }
        return entry != null ? entry.getKey() : null;
    }

    public BigDecimal getBestAsk() {
        long stamp = sl.tryOptimisticRead();
        Map.Entry<BigDecimal, PriceLevel> entry = asks.firstEntry();
        if (!sl.validate(stamp)) {
            stamp = sl.readLock();
            try { entry = asks.firstEntry(); }
            finally { sl.unlockRead(stamp); }
        }
        return entry != null ? entry.getKey() : null;
    }

    public BigDecimal getMidPrice() {
        BigDecimal bid = getBestBid();
        BigDecimal ask = getBestAsk();
        if (bid == null || ask == null) return null;
        return bid.add(ask).divide(BigDecimal.TWO, 8, java.math.RoundingMode.HALF_UP);
    }

    public BigDecimal getSpread() {
        BigDecimal bid = getBestBid();
        BigDecimal ask = getBestAsk();
        if (bid == null || ask == null) return null;
        return ask.subtract(bid);
    }

    public int getDepth(Order.Side side, int levels) {
        ConcurrentSkipListMap<BigDecimal, PriceLevel> book = side == Order.Side.BUY ? bids : asks;
        return (int) book.entrySet().stream().limit(levels)
            .mapToLong(e -> e.getValue().getTotalQuantity().longValue()).sum();
    }
}
```

---

### Price Level — FIFO Queue per Price

```java
import java.math.BigDecimal;
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.ReentrantLock;

public class PriceLevel {

    private final BigDecimal price;
    // LinkedList for O(1) add (tail) and remove (head) → FIFO = price-time priority
    private final Queue<Order> orders = new LinkedList<>();
    private BigDecimal totalQuantity  = BigDecimal.ZERO;
    private final ReentrantLock lock  = new ReentrantLock();

    public PriceLevel(BigDecimal price) {
        this.price = price;
    }

    public void addOrder(Order order) {
        lock.lock();
        try {
            orders.add(order);
            totalQuantity = totalQuantity.add(order.getRemainingQuantity());
        } finally { lock.unlock(); }
    }

    public boolean removeOrder(String orderId) {
        lock.lock();
        try {
            boolean removed = orders.removeIf(o -> {
                if (o.getId().equals(orderId)) {
                    totalQuantity = totalQuantity.subtract(o.getRemainingQuantity());
                    return true;
                }
                return false;
            });
            return removed;
        } finally { lock.unlock(); }
    }

    // Match against incoming aggressive order — returns list of fills
    public List<Fill> match(Order aggressor) {
        lock.lock();
        try {
            List<Fill> fills = new ArrayList<>();
            while (!orders.isEmpty() && aggressor.getRemainingQuantity().compareTo(BigDecimal.ZERO) > 0) {
                Order passive = orders.peek();
                BigDecimal fillQty = aggressor.getRemainingQuantity()
                    .min(passive.getRemainingQuantity());

                aggressor.applyFill(fillQty, price);
                passive.applyFill(fillQty, price);
                totalQuantity = totalQuantity.subtract(fillQty);

                fills.add(new Fill(passive.getId(), aggressor.getId(), fillQty, price));

                if (passive.isFullyFilled()) {
                    orders.poll(); // remove from front
                }
            }
            return fills;
        } finally { lock.unlock(); }
    }

    public boolean isEmpty()                 { return orders.isEmpty(); }
    public BigDecimal getTotalQuantity()     { return totalQuantity; }
    public BigDecimal getPrice()             { return price; }
    public int getOrderCount()              { return orders.size(); }

    public record Fill(String passiveId, String aggressorId,
                       BigDecimal quantity, BigDecimal price) {}
}
```

---

### Tying It Together — Order Service

```java
public class OrderService {

    private final OrderBook orderBook;
    private final OrderStateMachine fsm = new OrderStateMachine();
    private final Map<String, Order> orders = new ConcurrentHashMap<>();

    public OrderService(String symbol) {
        this.orderBook = new OrderBook(symbol);
    }

    public Order submitOrder(String symbol, Order.Side side, Order.Type type,
                             BigDecimal qty, BigDecimal price) {
        Order order = new Order(generateId(), symbol, side, type, qty, price);

        // Pre-trade risk check → transition or reject
        if (!passesRiskCheck(order)) {
            fsm.transition(order, OrderEvent.REJECT);
            return order;
        }

        fsm.transition(order, OrderEvent.SUBMIT); // NEW → PENDING_ACK
        orders.put(order.getId(), order);
        fsm.transition(order, OrderEvent.ACK);    // PENDING_ACK → ACKNOWLEDGED

        orderBook.addOrder(order, price);
        tryMatch(order);                           // attempt matching
        return order;
    }

    private void tryMatch(Order aggressor) {
        ConcurrentSkipListMap<BigDecimal, PriceLevel> oppositeSide =
            aggressor.getSide() == Order.Side.BUY ? orderBook.getAsks() : orderBook.getBids();

        for (Map.Entry<BigDecimal, PriceLevel> entry : oppositeSide.entrySet()) {
            if (aggressor.getRemainingQuantity().compareTo(BigDecimal.ZERO) <= 0) break;

            // Price check (limit order)
            if (aggressor.getType() == Order.Type.LIMIT) {
                boolean priceOk = aggressor.getSide() == Order.Side.BUY
                    ? aggressor.getLimitPrice().compareTo(entry.getKey()) >= 0
                    : aggressor.getLimitPrice().compareTo(entry.getKey()) <= 0;
                if (!priceOk) break;
            }

            List<PriceLevel.Fill> fills = entry.getValue().match(aggressor);
            for (PriceLevel.Fill fill : fills) {
                Order passive = orders.get(fill.passiveId());
                OrderEvent aggressorEvent = aggressor.isFullyFilled()
                    ? OrderEvent.FULL_FILL : OrderEvent.PARTIAL_FILL;
                OrderEvent passiveEvent = passive.isFullyFilled()
                    ? OrderEvent.FULL_FILL : OrderEvent.PARTIAL_FILL;

                // Only transition if not already terminal (concurrent cancel race)
                if (!aggressor.getState().isTerminal())
                    fsm.transition(aggressor, aggressorEvent);
                if (!passive.getState().isTerminal())
                    fsm.transition(passive, passiveEvent);
            }

            if (entry.getValue().isEmpty())
                oppositeSide.remove(entry.getKey()); // clean up empty level
        }
    }

    public boolean cancelOrder(String orderId) {
        Order order = orders.get(orderId);
        if (order == null || order.getState().isTerminal()) return false;
        fsm.transition(order, OrderEvent.CANCEL);
        return orderBook.cancelOrder(orderId);
    }

    private boolean passesRiskCheck(Order order) {
        // In production: check position limits, notional limits, symbol restrictions
        return order.getQuantity().compareTo(new BigDecimal("1000000")) < 0;
    }

    private String generateId() {
        return "ORD-" + System.nanoTime();
    }
}
```

---

### Top Interview Questions on These Topics

**Concurrency:**
- Why can't you use `volatile` for `counter++`? Walk me through the 3 ops.
- What is the happens-before relationship, and why does double-checked locking need `volatile`?
- When would you choose `StampedLock` over `ReadWriteLock`?
- What is false sharing and how does `@Contended` fix it?
- Why does the Disruptor use a ring buffer instead of a `BlockingQueue`?

**Order Book / FSM:**
- Why is `ConcurrentSkipListMap` better than `TreeMap` with a lock for the order book?
- How do you handle a cancel racing with a fill? (CAS + terminal state check)
- Why use `LinkedList` (Queue) inside a price level, not `ArrayList`?
- How does your FSM prevent invalid transitions under concurrent updates?
- What happens if the exchange sends a fill for a cancelled order?

Want me to continue with **FIX protocol + EMS design**, or **TWAP/VWAP algorithm implementation** next?