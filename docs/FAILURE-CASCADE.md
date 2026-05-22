# FAILURE CASCADE ANALYSIS

## System Overview

### Current SwiftEats Monolith

- Single Node.js Express server
- 1 CPU
- 4 GB RAM
- Single PostgreSQL database
- max_connections = 100
- No Redis cache
- No CDN
- No load balancer
- Synchronous payment processing
- No auto-scaling

---

# Section 1 — Traffic Simulation Math

## Notification Reach

- Total users notified = 180 million
- Click-through rate = 8%

### Active Users

14,400,000 users open the app within 60 seconds.

## API Requests Per User

Estimated API calls per user during first minute:

1. Home feed request
2. Restaurant list request
3. Restaurant menu request
4. Cart request

Average = 4 API calls/user

## Peak RPS Calculation

Peak RPS = (active users × API calls per user) / 60

= (14,400,000 × 4) / 60

= 960,000 RPS

---

# Section 2 — Component Capacity Numbers

## Node.js Capacity

Single Node.js instance on t3.medium:

- Approximate max throughput = 12,000–15,000 RPS
- Event loop begins saturating after callback queue buildup
- Heap memory limit = 4 GB
- OOM likely around 15,000 queued requests

## PostgreSQL Capacity

Configured max_connections = 100

### Payment Call Hold Time

Razorpay API latency:

- Best case = 200ms
- Worst case = 2000ms

Database connection remains occupied during payment wait.

## Connection Pool Exhaustion Formula

Connections held at any moment:

(% non-payment RPS × query_time_s)
+
(% payment RPS × payment_hold_time_s)

Pool exhaustion occurs when active connections reach 100.

---

# Section 3 — Failure Cascade

## Failure 1 — PostgreSQL Connection Pool Exhaustion (CRITICAL)

### Trigger Point

Connection pool reaches 100 active connections.

### User Impact

- API requests begin timing out
- Orders fail
- Checkout freezes

### Next Failure Triggered

Node.js request queue backlog increases rapidly.

---

## Failure 2 — Node.js Event Loop Saturation (CRITICAL)

### Trigger Point

RPS exceeds 15,000 on single Node.js process.

### User Impact

- High latency
- Delayed responses
- 502 and 503 errors

### Next Failure Triggered

Memory exhaustion and process crash.

---

## Failure 3 — Synchronous Payment Amplification (HIGH)

### Trigger Point

Large payment concurrency during promo spike.

### User Impact

- Payment delays
- Duplicate retries
- Stuck checkout flow

### Next Failure Triggered

Database connection exhaustion worsens.

---

## Failure 4 — Promo Code Race Condition (HIGH)

### Trigger Point

Simultaneous promo redemption attempts.

### User Impact

- Duplicate coupon usage
- Incorrect discounts
- Inconsistent order state

### Next Failure Triggered

Database write contention.

---

## Failure 5 — Static Asset NIC Saturation (MEDIUM)

### Trigger Point

Millions of image requests hit same Node.js server.

### User Impact

- Slow image loading
- API latency spike
- Timeouts

### Next Failure Triggered

Event loop congestion.

---

# Section 4 — Incident Timeline

| Time | Event |
|------|------|
| T+0s | Push notification sent |
| T+5s | Traffic spike begins |
| T+10s | PostgreSQL connections approach limit |
| T+20s | Node.js event loop backlog increases |
| T+40s | Payment API latency spikes |
| T+1m | Users receive 500/503 errors |
| T+5m | Node.js process OOM crash |
| T+30m | Emergency mitigation begins |
| T+2h | Partial recovery completed |