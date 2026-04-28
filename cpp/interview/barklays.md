This is an excellent role for your background, Li. The **Fixed Income eTrading Platform Engineer - VP** role at Barclays is a near-perfect match for your deep experience in low-latency systems, FIX connectivity, and real-time data processing.

However, the job description (JD) has a specific emphasis on **Java, concurrency, microservices, and Fixed Income (specifically Repo)** , whereas your resume leans heavily into C++ and FX/Equities. The online assessment will probe both your **technical hard skills** (Java, concurrency, systems design) and your **behavioral/cognitive soft skills** (VP-level leadership, risk management, stakeholder management).

Below are **100 questions with answers and examples**, mapped to Barclays' VP expectations, your resume, and the specific JD.

---

### PART 1: CORE JAVA & CONCURRENCY (15 Questions)

*For the "Deep Java engineering experience (including concurrency/multi-threading)" requirement.*

**1. Q: Explain how you would implement a thread-safe, low-latency order cache in Java for an eTrading platform.**
- **A:** I would avoid `synchronized` blocks. Instead, use `ConcurrentHashMap` and `AtomicReference` for non-blocking operations. For a fixed-size cache, I'd use `LongAdder` for counters and `ReadWriteLock` only if reads vastly outnumber writes. Example: At BNP Paribas, I built a market data order book using `ConcurrentSkipListMap` for price levels, which provides O(log n) access without full map locking.

**3. Q: What is false sharing, and how did you detect/mitigate it in a trading system?**
- **A:** False sharing occurs when two threads modify different variables that reside on the same CPU cache line, forcing cache invalidation. I mitigated this at Mizuho by padding cache lines with `@Contended` (Java 8+) or adding dummy `long` fields between counters. We detected it via Linux `perf` stats showing high cache-miss rates even with low CPU usage.

**5. Q: How does the G1 garbage collector behave differently from ZGC in a low-latency trading app?**
- **A:** G1 is region-based but can still have "stop-the-world" pauses for mixed collections. ZGC is designed for sub-10ms pauses even with multi-terabyte heaps. For a VP-level repo trading system, I'd use ZGC with `-XX:+UseZGC` and tune `ConcGCThreads` to match core count. At Bloomberg FXGO, we avoided GC entirely by off-heap caching with Chronicle Map.

**7. Q: Write pseudocode for a rate limiter for incoming FIX messages (logical example).**
- **A:** Use a `ScheduledExecutorService` with a token bucket. Pseudocode:
  ```
  class FixRateLimiter:
      tokens = AtomicLong(maxTokens)
      refillRate = 1000  // per second
      scheduleAtFixedRate: tokens.set(min(maxTokens, tokens.get() + refillRate))
      allow(): return tokens.decrementAndGet() >= 0
  ```
  At Citigroup, we applied this per-session ID to prevent a single client from flooding the OMS.

**9. Q: Explain the difference between `submit()` and `execute()` on `ExecutorService`.**
- **A:** `execute()` takes a `Runnable` and returns nothing; exceptions are lost unless handled inside. `submit()` returns a `Future<T>` and exceptions are captured and re-thrown when `Future.get()` is called. For a FIX acceptor, I'd use `submit()` for order processing so the caller can block on the result with a timeout.

**11. Q: How do you debug a production deadlock in a Spring Boot microservice?**
- **A:** First, `kill -3 <pid>` to dump thread stacks (or `jstack`). Look for threads in `BLOCKED` state owning the same lock. Example: At Mizuho, we had a deadlock between a Kafka consumer thread and a session cleanup thread both locking a FIX session map. We fixed by reordering lock acquisition: always acquire session lock first, then order lock.

**13. Q: What is the volatile keyword's guarantee in Java?**
- **A:** Volatile ensures visibility (writes are flushed to main memory immediately) and happens-before ordering, but not atomicity. For a "running" flag, it's perfect. For a sequence number, use `AtomicLong`. At E*TRADE, we used volatile for a `shutdownRequested` flag across multiple FIX engine threads.

**15. Q: How would you implement a non-blocking, thread-safe order ID generator?**
- **A:** Use `AtomicLong` with `incrementAndGet()`. For distributed systems (multiple app instances), prepend a node ID: `(nodeId << 48) | AtomicLong.getAndIncrement()`. At BNP, we used a Snowflake-like generator to guarantee unique IDs across 10 gateway instances without coordination.

---

### PART 2: SPRING BOOT & MICROSERVICES (12 Questions)

*For "Spring Boot, microservices, API engineering, CI/CD."*

**2. Q: How do you manage distributed transactions across multiple microservices (e.g., order → risk → execution)?**
- **A:** Avoid 2PC (two-phase commit) in low-latency systems. Use the Saga pattern with choreography (events via Kafka). Example: At Mizuho, an "OrderCreated" event triggered risk check; risk approval published "OrderValidated" → execution. Each step logs compensation actions (e.g., "OrderCanceled") in a database. We used Spring State Machine to track saga status.

**4. Q: What is Hystrix or Resilience4j, and how did you use it in a trading context?**
- **A:** These provide circuit breakers, bulkheads, and retries. At Bloomberg, we wrapped calls to an external FIX gateway with a circuit breaker: after 5 failures in 10 seconds, the circuit opens and requests fail fast for 30 seconds, then half-open. This prevented cascading failures when one bank's session dropped.

**6. Q: Explain Spring Boot's auto-configuration – how would you disable a specific one?**
- **A:** Auto-configuration is conditional on classpath beans. To disable, use `@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})` or set `spring.autoconfigure.exclude` in `application.yml`. At Barclays, I'd exclude `RedisAutoConfiguration` if using Hazelcast for caching instead.

**8. Q: How do you handle API versioning in a RESTful eTrading frontend?**
- **A:** Use URI versioning: `/v1/orders` vs `/v2/orders`. At Citigroup, we also used content negotiation via `Accept` header (`application/vnd.barclays.v2+json`). We maintained both versions for 2 release cycles, with `@RequestMapping` headers in Spring MVC. Deprecated endpoints logged warnings and returned `Deprecation` header.

**10. Q: What is the purpose of Spring Cloud Gateway vs Zuul vs Netflix Ribbon?**
- **A:** Gateway is modern, non-blocking (WebFlux) for routing. Zuul 1.x is blocking. Ribbon is client-side load balancing (now replaced by Spring Cloud LoadBalancer). For a repo trading platform, I'd use Gateway with circuit breakers and route predicates based on `X-Client-ID` header to route different liquidity providers to different backend clusters.

**12. Q: How do you secure Spring Boot microservices internally?**
- **A:** Use OAuth2 with JWT tokens (Keycloak or Okta). For internal service-to-service, use mTLS (mutual TLS) and a service mesh like Istio. At Mizuho, we used Spring Security with `@PreAuthorize("hasRole('TRADER')")` on controller methods, and an API gateway validated tokens before forwarding.

---

### PART 3: FIX PROTOCOL & FIXED INCOME / REPO (12 Questions)

*For "FIX connectivity, D2C/D2D, and repo/fixed income financing."*

**2. Q: Explain the difference between FIX 4.2 and FIX 5.0 SP2 for fixed income trading.**
- **A:** FIX 5.0 introduces repeating groups more cleanly, the `ApplicationSequenceControl` block for recovery, and component blocks to reduce message bloat. For repos, FIX 5.0 has specific fields for `RepoTerm` (tag 226), `RepoType` (tag 227), and `AgreementDesc` (tag 913). At Mizuho, we migrated from 4.2 to 5.0 to support complex equity swap terms.

**4. Q: What is a Drop Copy session, and how did you implement one?**
- **A:** Drop Copy provides a real-time, read-only feed of executed orders for middle-office/risk systems. At Mizuho, I built a Drop Copy pipeline for equity swaps: a custom Java adapter subscribed to the internal FIX engine's execution reports (MsgType 8, 35=8) and republished them over a separate FIX session to Murex MX.3. We ensured sequence number integrity by persisting every message to a Kafka topic.

**6. Q: How would you handle sequence number reset on a FIX session after a disconnect?**
- **A:** The initiator sends `Logon (35=A)` with `ResetSeqNumFlag (tag 141) = Y`. The acceptor must verify its local sequence and respond. However, for regulatory compliance, we never reset on production without audit. Instead, at E*TRADE, we stored last outgoing seqnum in Redis before each message, and on reconnect we sent `ResendRequest (35=2)` from last known inbound seqnum.

**8. Q: Describe the life cycle of a repo trade from FIX message perspective.**
- **A:** 1) `NewOrderSingle (35=D)` with RepoTerm, AgreeDt, and EndSegDt. 2) `ExecutionReport (35=8)` with ExecType=0 (New). 3) For open repo, periodic `TradeCaptureReport (35=AE)` for mark-to-market. 4) At maturity, a `SettlementInstruction (35=AI)` or a closing `OrderCancelReplaceRequest`. The Barclays JD mentions "securities finance" – we'd also interface with DTCC to send repo settlement instructions via FIXML.

**10. Q: How do you test a FIX engine without a counterparty?**
- **A:** Use VeriFix (as I did at E*TRADE) or QuickFIX/J's `Acceptor` and `Initiator` in the same JVM. I built a simulation environment that could generate every FIX message type, including out-of-order sequences and gap fills. At BNP Paribas, we used a Python script with `simplefix` to replay capture files from production to validate new feed handlers.

**12. Q: What are the key fields in a FIX Trade Capture Report for settlement?**
- **A:** `TradeReportID (571)`, `TradeID (1003)`, `SecondaryTradeID (1040)`, `SettlCurrAmt (119)`, `SettlDate (64)`, `SendingTime (52)`, `LastMkt (30)`. For DTCC CTM alignment (as in my Mizuho project), we also needed `ExecBroker (76)`, `ClearingFirm (439)`, and `SettlInstID (162)`.

---

### PART 4: DATA PIPELINES, MESSAGING & RESUME-SPECIFIC (12 Questions)

*Based on your Kafka, Snowflake, PySpark, Databricks experience.*

**2. Q: How would you design a real-time trade enrichment pipeline with Kafka?**
- **A:** Use Kafka Streams or Spark Structured Streaming. At Mizuho, I built: Input topic `raw_trades` → Transformer (lookup reference data from Redis cache) → Flatten JSON → Output topic `enriched_trades`. Then a Snowflake sink connector wrote to a staging table. Exactly-once semantics via `processing.guarantee=exactly_once` and idempotent producer.

**4. Q: Explain the difference between ETL and ELT. Which do you prefer for a trade reporting system?**
- **A:** ETL transforms before loading (good for sensitive data or weak target DB). ELT loads raw then transforms (better for cloud warehouses like Snowflake). For my Mizuho project, we used ELT: ingested raw JSON into Snowflake's `VARIANT` column, then used PySpark jobs for heavy transformations. This let us reprocess without reloading.

**6. Q: How did you use AI (Claude/Copilot/Gemini) to boost productivity?**
- **A:** 1) Generating boilerplate Spring Boot REST controllers from OpenAPI specs. 2) Writing PySpark UDFs for date parsing – saved 4 hours. 3) Converting a legacy C++ order validator to Java with unit tests. 4) Debugging a Kafka rebalance issue: I pasted logs into Claude, and it suggested `max.poll.interval.ms` tuning. 5) Generating Javadoc and OpenAPI annotations automatically.

**8. Q: What is the downside of using PySpark on Databricks for real-time trading?**
- **A:** PySpark has higher latency than Kafka Streams due to JVM-GC interaction and serialization overhead. For sub-100ms requirements, use Java Spark or Flink. At Mizuho, we used PySpark only for near-real-time analytics (5-minute windows) and Java for sub-second order processing.

**10. Q: How would you manage schema evolution in a Kafka topic?**
- **A:** Use Avro with a Schema Registry. Backward compatibility: add only optional fields, never delete required. At Bloomberg, we tagged each message with a version field. Our `@KafkaListener` used a custom deserializer that could read v1 and v2 into a common Java POJO with defaults. Breaking changes required a new topic.

**12. Q: You mentioned Tidal Automation – walk me through a workflow you automated.**
- **A:** At Mizuho, daily BAU included: 1) SFTP DTCC settlement files, 2) Validate FIX session health, 3) Generate trade reconciliation report. I built a Tidal job chain: a Python script for SFTP → if success, trigger a REST call to a Spring Boot health check → if success, run Snowflake query → email results. This reduced operational risk and saved 2 hours daily.

---

### PART 5: BEHAVIORAL & VP LEADERSHIP (20 Questions)

*For "Vice President Expectations – lead, influence, manage risk, stakeholders."*

**2. Q: Tell me about a time you had to push back on a trader's request.**
- **A:** A trader at Bloomberg wanted to bypass our risk checks for a "latency-sensitive" client. I explained that this violated our risk mandate and the SEC's best-execution rules. Instead, I proposed optimizing the risk checks from 5ms to 0.5ms by moving them in-memory with a pre-computed portfolio snapshot. The trader agreed. Outcome: 80% latency reduction without compliance breach.

**4. Q: How do you manage technical debt while delivering new features?**
- **A:** I use the "boy scout rule" – leave code better than you found it. At Citigroup, we allocated 20% of each sprint to debt (refactoring the FIX message parser, improving logging). For major debt (e.g., legacy CORBA system), I built a business case: "Cost of delay = $X/year in support, risk of Y outages." Got approval for a dedicated quarter.

**6. Q: Describe a conflict with a peer team (e.g., Operations, Risk).**
- **A:** At Mizuho, Operations wanted manual confirmation for every DTCC trade submission. I wanted fully automated. Conflict: they feared regulatory fines. Solution: I built a "supervisory dashboard" that showed each auto-submitted trade with a 10-second undo button. After 3 months with zero errors, they agreed to remove the manual step. Result: settlement time dropped from 45 min to 2 min.

**8. Q: How do you ensure your team follows secure coding practices?**
- **A:** 1) Mandatory pre-commit hooks for secrets scanning (truffleHog). 2) SonarQube rules for SQL injection, deserialization vulnerabilities. 3) Weekly threat modeling for new features (e.g., how could an attacker spoof a FIX session?). At Bloomberg, we also ran annual third-party penetration tests on our FIX gateways.

**10. Q: Tell me about a time you made a high-stakes technical decision that failed.**
- **A:** At BNP Paribas, I chose to implement a custom UDP multicast resend mechanism instead of using a vendor library. It worked for 6 months, then a Cisco switch upgrade caused packet reordering that our logic couldn't handle. I owned the mistake, rolled back to the vendor solution in 48 hours, and added end-to-end checksums. Lesson: don't reinvent proven protocols.

**12. Q: How do you communicate technical risks to non-technical stakeholders (e.g., Head of Repo Trading)?**
- **A:** Use business impact language, not tech jargon. Instead of "our Kafka consumer lag is 50,000 messages," say "trade confirmations are delayed by up to 30 seconds during peak volatility, risking DTCC fines of $10k/day." I use a simple RAG (Red/Amber/Green) dashboard. At Barclays, I'd show "Time to Last Trade" on a big screen.

**14. Q: Describe your experience mentoring junior engineers.**
- **A:** At Bloomberg, I mentored three associates. One struggled with concurrency. I paired with him for a week on a FIX sequence number handler, using code reviews and whiteboarding memory models. He later led a Kafka migration. I also started a "brown bag" series on low-latency Java patterns. My measure: two of them were promoted to Senior within 18 months.

**16. Q: How do you handle a production outage at 2 AM?**
- **A:** First, stabilize – rollback the last deployment or restart the service. Second, communicate – update the incident channel every 15 minutes, even if "still investigating." Third, after resolution, lead a blameless post-mortem within 48 hours. At Mizuho, our post-mortem on a FIX session hang led to a heartbeat monitoring improvement that prevented recurrence.

**18. Q: What is your approach to managing a strategic change (e.g., migrating from on-prem to cloud)?**
- **A:** 1) Inventory and categorize (critical vs. nice-to-have). 2) Create a phased roadmap with fallback triggers. 3) Build a cost-benefit model (e.g., cloud = $200k/year less hardware, but $100k more egress fees). 4) Pilot with a non-critical repo trade reporting service. At E*TRADE, we moved the FIX simulator first, then the gateway.

**20. Q: Why Barclays? And why this specific role?**
- **A:** Barclays is a top-tier fixed income house, and repos are a strategic growth area. My background spans Bloomberg (FX), Mizuho (equity swaps), and BNP (HFT) – all relevant to building a modern eTrading platform. I want to apply my Java + low-latency + FIX expertise to repo, which has unique challenges (term structures, collateral management). The VP remit of "setting strategy and influencing stakeholders" matches my career stage.

---

### PART 6: SYSTEM DESIGN & TRADING SCENARIOS (12 Questions)

**2. Q: Design a real-time PnL calculation engine for a repo desk.**
- **A:** Inputs: trades (FIX), daily funding rates (Bloomberg terminal), collateral values. Architecture: Kafka (trades) + Redis (current rates) + Flink (windowed calculations). Output: PnL per book per 5 minutes. For low latency, we'd pre-aggregate in-memory using `LongAccumulator` per trader. At Bloomberg FXGO, we did similar for FX swaps – key was handling intraday rate changes gracefully.

**4. Q: How would you reduce the latency of a FIX order from 5ms to 500µs?**
- **A:** 1) Move from HTTP to binary FIX over TCP. 2) Use direct byte buffers (off-heap) to avoid GC. 3) Collocate with matching engine. 4) Bypass kernel for UDP market data (PF_RING). 5) Pin processing threads to CPU cores. At BNP Paribas, we achieved 150µs by eliminating JSON parsing and using pre-allocated fix-length structs.

**6. Q: Design a failover system for a FIX gateway.**
- **A:** Active-passive with heartbeat. Primary publishes sessions and sequence numbers to Redis (every message). Passive subscribes to Redis and also consumes all market data to keep its book warm. On failure, passive reads last sequence from Redis, sends a `ResendRequest` for missing messages (if any), and sends a `Logon` with reset flag false. At E*TRADE, we added a "shadow" process that was read-only ready.

**8. Q: How would you integrate an external repo trading platform (e.g., TradeWeb or BrokerTec)?**
- **A:** They typically support FIX or REST. I'd build a thin adapter: translate internal standard FIX (RepoTerm, AgreeDt) to BrokerTec's specific tags. Use a circuit breaker and retry with exponential backoff. Offload non-critical fields to headers. At Mizuho, we built an Ullink adapter in less than 4 weeks using Spring Boot and QuickFIX/J.

**10. Q: You are asked to replace a legacy monolith with microservices. Where do you start?**
- **A:** Identify the pain points (e.g., FIX session management is coupled with order routing). Extract the order routing service first – it has clearly defined boundaries. Use strangler pattern: new orders go to new service, old system still handles existing. At Citigroup, we replaced the order router incrementally – each asset class migrated one by one.

**12. Q: How would you monitor the health of a real-time trading system?**
- **A:** Four golden signals: latency (p99 order-to-ack), traffic (messages/sec), errors (FIX reject rate), saturation (CPU, GC time, network drops). Use Splunk (as I have) and Prometheus + Grafana. At Bloomberg, we also had synthetic orders: every 10 seconds, a test order was routed to a sandbox and back – if it took >50ms, page the team.

---

### PART 7: RISK, CONTROLS & COMPLIANCE (8 Questions)

*For "Manage and mitigate risks through assessment, control and governance."*

**2. Q: What is the difference between market risk and operational risk in eTrading?**
- **A:** Market risk = loss from price movements (e.g., repo rates spike). Operational risk = loss from failed processes, people, or systems (e.g., FIX session drops and you miss a trade). At VP level, you own operational risk controls: sequence number recovery, audit trails, and access controls. The JD emphasizes "strengthening controls" – I would implement automated trade reconciliation daily.

**4. Q: How do you ensure auditability of FIX trades for regulators (e.g., SEC Rule 606)?**
- **A:** Every FIX message (including heartbeats and rejections) must be persisted in a WORM (write-once, read-many) store with a timestamp from a stratum-1 NTP source. At Mizuho, we wrote every raw FIX message to AWS S3 with a retention of 7 years, and indexed metadata (OrderID, SenderCompID) in Snowflake for search.

**6. Q: Describe a time you found a control weakness and fixed it.**
- **A:** At BNP Paribas, I noticed that our market data feed handler logged every packet (PII-level data) to a file world-readable. This violated GDPR. I fixed by: 1) Redacting prices to tick-level only, 2) Changing log permissions to 640, 3) Implementing a monthly log rotation and audit. No further violations.

**8. Q: What is a pre-trade risk check, and how would you implement one?**
- **A:** Pre-trade risk validates that an order won't exceed credit limit, position limit, or price collar. In Java, I'd use a `ConcurrentHashMap` of trader → current net exposure, updated atomically. On each `New Order Single`, check `exposure + orderQty <= limit`. Reject with FIX ExecType=8 (Rejected). At Barclays repo desk, I'd add a haircut check based on collateral type.

---

### PART 8: BRAIN-TEASERS & QUICK LOGICAL REASONING (9 Questions)

*Common in finance online assessments.*

**2. Q: You have 100 FIX sessions, each sending 1000 msgs/sec. How many messages per second total? Follow-up: How many threads for optimal processing?**
- **A:** 100,000 msgs/sec. For threads: modern rule of thumb = number of cores (e.g., 32). Use a disruptor pattern or a thread pool per 10 sessions to reduce contention. At BNP, we had 48 cores, used 4 Kafka consumers with 12 threads each – gave best throughput without GC spikes.

**4. Q: Estimate the daily bandwidth used by a FIX adapter sending 5000 orders/sec, average 500 bytes/msg.**
- **A:** 5000 * 500 = 2.5 MB/sec. Times 86400 sec/day = 216 GB/day. Add 30% overhead for TCP and encryption → ~280 GB/day. This fits in a standard 10 Gbps link (which can do 108 TB/day). I'd still plan for failover.

**6. Q: You see a CPU spike to 100% on one core. How do you diagnose?**
- **A:** `top -H -p <pid>` to find the thread ID, then `jstack` to see what that thread is doing. Common culprits: busy-wait loop, tight while(true) with no sleep, or a polling FIX session. At Bloomberg, we found a GC thread thrashing – fixed by increasing heap and switching to G1.

**8. Q: Why is sorting faster on arrays than linked lists?**
- **A:** Arrays have cache locality – prefetching works. Linked lists cause pointer chasing and cache misses. For a market data order book (price levels), I'd use an array-backed tree (e.g., `ArrayList` of `PriceLevel`) even if resizing is needed.

---

### PART 9: BARCLAYS VALUES & MINDSET (8 Questions)

*Must align to "Respect, Integrity, Service, Excellence, Stewardship" and "Empower, Challenge, Drive."*

**2. Q: Give an example of "Stewardship" in your work.**
- **A:** At Mizuho, I refactored a legacy FIX parser that was undocumented and fragile. Instead of just rewriting, I documented every edge case, wrote regression tests, and then rewrote. I also trained two juniors to maintain it. That's stewardship – preserving and improving institutional knowledge.

**4. Q: How did you "Challenge" a decision constructively?**
- **A:** At Bloomberg, the team wanted to adopt a new message bus. I challenged: "What is the failure mode if the bus goes down?" They hadn't considered. I built a fault-injection test that proved the bus would drop heartbeats. We chose a different vendor. Challenging with data, not ego.

**6. Q: Describe a time you went "above and beyond" for a client (internal or external).**
- **A:** A client (hedge fund) at E*TRADE had an obscure FIX tag that wasn't supported for their algo. I spent a weekend writing a custom adapter that translated their tag into our internal schema. They stayed with us for 3 more years. That's "Service" and "Excellence."

**8. Q: How do you "Empower" your colleagues?**
- **A:** I build self-service tools. At Citigroup, I created a Jenkins pipeline with a "Deploy to UAT" button so that QA could test without asking engineering. I also rotated "on-call" shadowing so junior engineers learn incident response with safety. Empowerment = lowering friction.

---

### FINAL TIPS FOR THE ONLINE ASSESSMENT

| Assessment Type | Likely Format | Strategy |
| :--- | :--- | :--- |
| **Cognitive / Numerical** | Timed, multiple-choice (e.g., "Which bar chart shows 20% growth?" or "How many FIX messages in 3 mins?") | Use scrap paper. Pace yourself – skip and return. |
| **Situational Judgement** | "What would you do?" – options from passive to aggressive | Barclays values **collaboration, risk awareness, and escalation**. Never choose "Ignore" or "Handle in secret." Prefer "Discuss with lead" or "Escalate to risk." |
| **Java/Programming** | HackerRank or Codility: multi-threading, fix a bug in a FIX parser, implement a cache | Write clean, compilable code. Add comments. Handle edge cases (null, negative seq numbers). Mention unit tests. |
| **Personality** | "Most like me / Least like me" | Be consistent. For VP trading: high **conscientiousness**, **analytical**, **collaborative**, **resilient**. Avoid extreme "loner genius" or "always avoids risk." |

**Your resume is a weapon.** Every answer should implicitly reference a concrete accomplishment: "When I built the FIXML pipeline at Mizuho…" or "At BNP with ITCH feeds…" This proves you have done it, not just read about it.

Good luck, Li. You are exactly the kind of candidate Barclays needs for this role. Go demonstrate that.