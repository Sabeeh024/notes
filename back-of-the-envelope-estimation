# Back-of-the-Envelope Estimation (System Design)

Goal: get an answer within the right **order of magnitude** (10x), fast. Interviewers and designers care about your assumptions and method, not the exact digits. It tells you whether a design needs one server or a thousand, one DB or sharding, a CDN or not.

---

## 1. Core Terms

| Term | Meaning | Notes |
| --- | --- | --- |
| **MAU** | Monthly Active Users | Usually the number you are given |
| **DAU** | Daily Active Users | DAU = MAU × (fraction who use daily) |
| **RPS** | Requests per second | Requests hitting your servers |
| **QPS** | Queries per second | Queries hitting your database. One request can trigger several queries, so QPS ≥ RPS often |
| **TPS** | Transactions per second | Used for write/payment-style systems |
| **Peak factor** | Peak load ÷ average load | Design for peak, not average |
| **Read:Write ratio** | e.g. 100:1 for social feeds | Decides caching, replicas, DB choice |

Key idea: **read vs write behaviour changes what "an active user" generates.** A DAU may read 100 posts but write 0.5. Always estimate reads and writes separately.

---

## 2. The Standard Method

1. **Clarify scope**: which features? Reads, writes, or both? Global or regional?
2. **State assumptions** (MAU, % daily active, actions per user per day, object sizes). Write them down and round them.
3. **Per-day volume**: DAU × actions per user.
4. **Per-second average**: daily volume ÷ 86,400 (use ~10^5 for quick math).
5. **Peak**: average × peak factor (commonly 2x to 5x; ask or state it).
6. **Derive**: storage, bandwidth, memory/cache, server count.
7. **Sanity-check** the result against real-world intuition.

---

## 3. Scientific Notation Technique

Avoid juggling long numbers. Convert to `a × 10^n`, multiply the coefficients, add the exponents.

- 150M = 1.5 × 10^8
- Seconds per day = 86,400 ≈ 10^5 (more precisely 8.64 × 10^4)
- Multiply coefficients, add/subtract powers of ten.

**Round to friendly numbers**: 86,400 → 100,000 for fast math. This overestimates the divisor by about 14%, so the answer comes out slightly low. When precision matters, use 8.64 × 10^4, or just use ~90,000.

### Reference: Number Units

| Number unit | Value | Scientific | Byte unit |
| --- | --- | --- | --- |
| Thousand (K) | 1,000 | 10^3 | KB |
| Million (M) | 1,000,000 | 10^6 | MB |
| Billion (B) | 1,000,000,000 | 10^9 | GB |
| Trillion (T) | 10^12 | 10^12 | TB |
| Quadrillion | 10^15 | 10^15 | PB |

### Reference: Powers of Two

| Power | Approx. | Name |
| --- | --- | --- |
| 2^10 | 1 thousand | 1 KB |
| 2^20 | 1 million | 1 MB |
| 2^30 | 1 billion | 1 GB |
| 2^40 | 1 trillion | 1 TB |
| 2^50 | 1 quadrillion | 1 PB |

### Time Shortcuts

| Period | Seconds | Approx. |
| --- | --- | --- |
| 1 minute | 60 | 6 × 10^1 |
| 1 hour | 3,600 | 3.6 × 10^3 |
| 1 day | 86,400 | ~10^5 |
| 1 month | ~2.6 million | ~2.5 × 10^6 |
| 1 year | ~31.5 million | ~3 × 10^7 |

---

## 4. Worked Example: Tweets Created per Second

*(Fabricated numbers, for practice.)*

**Assumptions**
- 300M MAU
- 50% use it daily → DAU = 300M × 0.5 = **150M**
- 25% of DAU post, each posts 2 tweets/day → 0.25 × 2 = **0.5 tweets per DAU per day**
- Peak = **2x** average (people tweet when they wake up)

**Daily volume**
- 150M × 0.5 = **75M tweets/day**

**Average per second**
- 75M ÷ 86,400 ≈ **~870 tweets/s**

**Peak per second**
- 870 × 2 ≈ **~1,740 tweets/s**

**Same thing in scientific notation**
```
1.5×10^8 × 0.5 × 2 / 10^5 = (1.5 × 0.5 × 2) × 10^(8-5) = 1.5 × 10^3 ≈ 1,500/s
```
Using 10^5 for 86,400 gives 1,500 vs. the precise 1,736. Same order of magnitude, which is the point.

---

## 5. Storage Estimation

```
Storage per day  = writes per day × size per item
Storage per year = per day × 365   (≈ ×400 for quick math)
Total with retention = per year × years × replication factor
```

**Example: tweet storage**
- 75M tweets/day × ~300 bytes of text/metadata ≈ 22.5 GB/day
- × 365 ≈ ~8 TB/year (text only)
- With media: assume 10% of tweets have a 500 KB image → 7.5M × 500 KB ≈ 3.75 TB/day. Media dominates, so it goes in object storage plus a CDN, not the main DB.
- Multiply by the **replication factor** (typically 3).

### Typical Object Sizes (rules of thumb)

| Item | Approx. size |
| --- | --- |
| A character (ASCII/UTF-8) | 1 to 4 bytes |
| Short text post/metadata row | 0.2 to 1 KB |
| User profile record | 1 to 2 KB |
| Compressed photo (web) | 200 KB to 2 MB |
| Short video (1 min, compressed) | 5 to 50 MB |
| 1 hour of HD video | ~1 to 3 GB |
| UUID / 64-bit ID | 16 / 8 bytes |

---

## 6. Bandwidth Estimation

```
Bandwidth = (requests per second) × (average response size)
```
- Ingress (uploads/writes) and egress (downloads/reads) are separate. Egress is usually much larger for read-heavy systems.
- Example: 10,000 reads/s × 100 KB = 1 GB/s egress → CDN is mandatory.
- Convert bytes to bits (× 8) when comparing to network link speeds.

---

## 7. Memory / Cache Estimation

- Common heuristic: **80/20 rule**, i.e. cache the hottest ~20% of data to serve ~80% of reads.
- Cache size ≈ daily read volume × object size × 0.2.
- One Redis/Memcached box typically holds tens to a few hundred GB.

---

## 8. Server Count Estimation

```
Servers = peak RPS / RPS capacity per server
```

Rough per-server capacity (varies hugely by workload):

| Server type | Typical capacity |
| --- | --- |
| Simple web/API server | ~1,000 to 10,000 RPS |
| CPU-heavy / heavy business logic | ~100 to 1,000 RPS |
| One relational DB node | ~1,000 to 10,000 QPS (simple queries) |
| Redis (single node) | ~50,000 to 100,000+ ops/s |

Add headroom (target ~50 to 70% utilisation) and redundancy (N+1 or N+2) for failures and deployments.

---

## 9. Latency Numbers Worth Knowing

Approximate (originally popularised by Jeff Dean; hardware has improved but the ratios still hold):

| Operation | Approx. time |
| --- | --- |
| L1 cache reference | ~1 ns |
| Main memory (RAM) reference | ~100 ns |
| Compress 1 KB | a few µs |
| Read 1 MB sequentially from RAM | ~10s of µs |
| SSD random read | ~100 µs |
| Round trip within same data center | ~0.5 ms |
| Read 1 MB sequentially from SSD | ~0.1 to 1 ms |
| HDD seek | ~5 to 10 ms |
| Cross-continent round trip | ~100 to 150 ms |

Takeaways: memory is far faster than disk, disk far faster than network across regions, sequential beats random, and compress before sending over long distances.

---

## 10. Availability ("Nines")

| Availability | Downtime per year | Per month |
| --- | --- | --- |
| 99% (two nines) | ~3.65 days | ~7.3 hours |
| 99.9% | ~8.76 hours | ~43.8 minutes |
| 99.99% | ~52.6 minutes | ~4.4 minutes |
| 99.999% | ~5.3 minutes | ~26 seconds |

---

## 11. Peak Factors and Read/Write Skew

- **Peak factor**: choose by usage pattern. Examples: maps/commute apps 3x to 5x; global social apps 2x; events/sales (ticketing, flash sales) can be 10x to 100x+.
- **Read-heavy** (feeds, product pages): ratios of 10:1 to 1000:1. Favour caching, read replicas, CDN.
- **Write-heavy** (logging, metrics, IoT): favour append-only stores, queues, sharding, batching.
- Also account for **fan-out**: one write can cause many reads/writes (e.g. a celebrity tweet delivered to millions of timelines).

---

## 12. Worked Example 2: URL Shortener

**Assumptions**: 100M new URLs/month, read:write = 100:1, each record ~500 bytes, keep 5 years.

- Writes/s: 100M ÷ (2.5 × 10^6 s) ≈ **40 writes/s**
- Reads/s: 40 × 100 = **4,000 reads/s** (peak at 2x ≈ 8,000)
- Records in 5 years: 100M × 12 × 5 = **6 billion**
- Storage: 6B × 500 B = **3 TB** (before replication)
- Short code length: base62 with 7 chars = 62^7 ≈ 3.5 trillion combinations, comfortably above 6B.
- Cache: 20% of daily reads. 4,000/s × 86,400 ≈ 350M reads/day × 500 B × 0.2 ≈ **35 GB**, which fits in one cache node.

Conclusion: moderate load. A few app servers, a replicated DB (sharding optional), and a cache layer are enough.

---

## 13. Worked Example 3: Video Streaming Storage

**Assumptions**: 500K video uploads/day, average 5 min, ~50 MB stored per video after transcoding to one rendition.

- 500K × 50 MB = 25 TB/day
- × 365 ≈ ~9 PB/year
- Multiple renditions (e.g. 3x) and replication (3x) can push this toward ~80 PB/year → object storage, tiering of cold content, CDN for delivery.

---

## 14. Common Mistakes

- Forgetting to separate reads and writes.
- Using average instead of peak load.
- Mixing units (bits vs bytes, MB vs MiB, per day vs per second).
- Ignoring replication, backups, indexes (these multiply storage by 2x to 5x).
- Over-precision: false accuracy wastes time. Round aggressively.
- Not sanity-checking the answer (e.g. "1 billion RPS" for a mid-size app is a red flag).
- Forgetting media/blobs, which usually dwarf text data.

---

## 15. Quick Cheat Sheet

```
DAU            = MAU × daily-use %
Daily ops      = DAU × ops per user per day
Avg per second = daily ops / 86,400   (≈ /10^5)
Peak per sec   = avg × peak factor (2x to 5x)
Storage/day    = writes/day × item size
Storage total  = per day × 365 × years × replicas
Bandwidth      = RPS × response size
Servers        = peak RPS / per-server RPS (+ headroom + redundancy)
Cache          ≈ 20% of daily read data
```

**Habit checklist**: clarify → assume → round → scientific notation → peak → storage → bandwidth → servers → sanity check.

---

## 16. Related Topics to Study Next

- **Capacity planning & autoscaling**: horizontal vs vertical scaling, load balancers
- **Caching strategies**: cache-aside, write-through, eviction (LRU/LFU), CDN
- **Database scaling**: indexing, replication, sharding/partitioning, SQL vs NoSQL tradeoffs
- **Consistency & CAP theorem**, eventual consistency
- **Message queues & async processing** (Kafka, RabbitMQ) for absorbing write spikes
- **Rate limiting & backpressure**
- **Back-of-envelope in practice**: load testing to replace guesses with measurements
- **SLA / SLO / SLI** and error budgets
- **Data modelling & ID generation** (snowflake IDs, base62 encoding)
- **Classic design problems to practise on**: URL shortener, Twitter/news feed, chat app, YouTube, ride-hailing, rate limiter, notification system
