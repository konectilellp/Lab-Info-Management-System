
# 🧠 1. WHY KAFKA IS CENTRAL IN YOUR SYSTEM

Without Kafka:

* Services call each other → tight coupling → fragile system

With Kafka:

* Services react to events → **loosely coupled, scalable, resilient**

👉 In your LIS:

> Kafka = **central nervous system**

---

# 🧱 2. EVENT-DRIVEN ARCHITECTURE OVERVIEW

### Pattern:

```text
Producer → Kafka Topic → Consumer Group → Service Action
```

### Example:

```text
Order Service → "order.created" → Kafka → Sample Service consumes
```

---

# 🧭 3. TOPIC DESIGN (VERY IMPORTANT)

Design topics by **domain events**, not services.

---

## 🧾 Core Topics

### 1. Order Domain

```text
orders.events
```

Events:

* `order.created`
* `order.cancelled`

---

### 2. Sample Domain

```text
samples.events
```

Events:

* `sample.collected`
* `sample.received`
* `sample.rejected`

---

### 3. Results Domain

```text
results.events
```

Events:

* `result.received` (from machine)
* `result.validated`

---

### 4. Reports Domain

```text
reports.events
```

Events:

* `report.generated`
* `report.approved`

---

### 5. Billing Domain

```text
billing.events
```

Events:

* `invoice.created`
* `payment.completed`

---

### 6. Workflow Engine

```text
workflow.events
```

Events:

* `workflow.started`
* `step.completed`
* `workflow.completed`

---

### 7. Instrument Integration

```text
instrument.raw
instrument.processed
```

---

# ⚙️ 4. EVENT STRUCTURE (STANDARD CONTRACT)

---

## ✅ Base Event Schema (ALL EVENTS)

```json
{
  "event_id": "uuid",
  "event_type": "order.created",
  "timestamp": "ISO8601",
  "tenant_id": "uuid",
  "source": "order-service",
  "version": "v1",
  "data": { }
}
```

---

## Example: `order.created`

```json
{
  "event_id": "uuid",
  "event_type": "order.created",
  "tenant_id": "t1",
  "data": {
    "order_id": "o123",
    "patient_id": "p123",
    "tests": ["cbc", "lft"]
  }
}
```

---

## Example: `result.received`

```json
{
  "event_type": "result.received",
  "data": {
    "sample_id": "s123",
    "machine_id": "m1",
    "values": [
      {"parameter": "HB", "value": "13.5"}
    ]
  }
}
```

---

# 🔑 5. PARTITION STRATEGY (CRITICAL FOR SCALE)

---

## Rule:

Partition by **entity key**

---

### Recommended Keys:

| Topic           | Partition Key |
| --------------- | ------------- |
| orders.events   | order_id      |
| samples.events  | sample_id     |
| results.events  | sample_id     |
| workflow.events | instance_id   |

---

## Why?

* Ensures **ordering per entity**
* Enables parallel processing

---

# 👥 6. CONSUMER GROUP DESIGN

---

## Each service = consumer group

Example:

| Service              | Topic Subscribed |
| -------------------- | ---------------- |
| Sample Service       | orders.events    |
| Workflow Engine      | samples.events   |
| Reporting Service    | results.events   |
| Notification Service | reports.events   |

---

## Scaling:

* Multiple consumers in same group
* Kafka distributes partitions

---

# 🔄 7. EXACTLY-ONCE / IDEMPOTENCY

---

## Problem:

Kafka = at-least-once delivery → duplicates possible

---

## Solution:

### Idempotency Table

```sql
processed_events (
  event_id UUID PRIMARY KEY,
  processed_at TIMESTAMP
)
```

---

### Logic:

```text
If event_id exists → skip
Else → process + store
```

---

# 🔁 8. RETRY & FAILURE HANDLING

---

## Strategy:

### 1. Retry Topic

```text
results.retry
```

### 2. Dead Letter Queue (DLQ)

```text
results.dlq
```

---

## Flow:

```text
Failure → Retry → Retry → DLQ
```

---

# ⏱️ 9. EVENT TIMING & SLA

---

Track:

* Event lag
* Processing time
* Step delays

---

## Example:

```text
sample.received → result.received → report.generated
```

---

# 🔗 10. EVENT CHOREOGRAPHY (REAL FLOW)

---

## Full Lab Flow:

```text
1. order.created
2. sample.collected
3. sample.received
4. result.received
5. result.validated
6. report.generated
7. report.approved
8. payment.completed
```

---

## Services React:

* Workflow Engine → listens to ALL
* Notification Service → listens to report/payment
* Analytics → listens to everything

---

# 🧠 11. SCHEMA MANAGEMENT (VERY IMPORTANT)

---

## Use:

* Schema Registry (Confluent / Apicurio)

---

## Benefits:

* Version control
* Backward compatibility
* Validation

---

## Example Evolution:

v1:

```json
{"order_id": "123"}
```

v2:

```json
{"order_id": "123", "priority": "urgent"}
```

---

# 🔐 12. SECURITY

* Encrypt data in transit (TLS)
* ACLs per topic
* Tenant isolation (important)

---

# ⚡ 13. PERFORMANCE OPTIMIZATION

---

## Settings:

* Batch size tuning
* Compression (Snappy)
* Retention policies

---

## Retention Example:

```text
instrument.raw → 1 day
orders.events → 7 days
audit.events → 30 days
```

---

# 🧠 14. ADVANCED PATTERNS (WHAT MAKES YOU ELITE)

---

## ✅ Event Sourcing (Optional but powerful)

* Store events as source of truth

---

## ✅ CQRS

* Separate read/write models

---

## ✅ Saga Pattern (Distributed transactions)

Example:

```text
Order → Payment → Confirmation
```

---

## ✅ Stream Processing

* Real-time analytics (Kafka Streams / Flink)

---

# 🚨 15. COMMON MISTAKES

* ❌ Topic per service (wrong abstraction)
* ❌ No schema versioning
* ❌ No idempotency
* ❌ Large messages (keep small)
* ❌ Ignoring DLQ

---

