# ARCHITECTURE REDESIGN

# Section 1 — Current Monolith Architecture

## Existing Architecture Diagram

                    Users
                       |
                       v
            +-------------------+
            |   Node.js Server  |
            | 1 CPU / 4 GB RAM  |
            +-------------------+
                       |
                       v
            +-------------------+
            |   PostgreSQL DB   |
            | max_connections=100 |
            +-------------------+

## Current Weaknesses

- Single point of failure
- No load balancing
- No caching layer
- No CDN
- Synchronous payment flow
- Database connection exhaustion risk
- Event loop saturation under traffic spikes
- Static assets served from application server

---

# Section 2 — Redesigned Scalable Architecture

## Proposed Architecture Diagram

                           Users
                              |
                              v
                   +------------------+
                   |   CloudFront CDN |
                   +------------------+
                              |
                              v
                   +------------------+
                   | Application Load |
                   |     Balancer     |
                   +------------------+
                              |
          -----------------------------------------
          |                  |                    |
          v                  v                    v

 +----------------+ +----------------+ +----------------+
 | Node.js API-1  | | Node.js API-2  | | Node.js API-N  |
 +----------------+ +----------------+ +----------------+

          |                  |                    |
          -----------------------------------------
                              |
                              v

                   +------------------+
                   |   Redis Cache    |
                   +------------------+

                              |
                              v

                   +------------------+
                   |    PgBouncer     |
                   +------------------+

                              |
                 --------------------------
                 |                        |
                 v                        v

      +------------------+      +------------------+
      | PostgreSQL Primary|      | Read Replica DB |
      +------------------+      +------------------+

                              |
                              v

                   +------------------+
                   |      SQS Queue   |
                   +------------------+

                              |
                              v

                   +------------------+
                   | Payment Workers  |
                   +------------------+

                              |
                              v

                   +------------------+
                   |    Razorpay API  |
                   +------------------+

---

# Section 3 — Component Explanations

## CloudFront CDN

Purpose:
- Cache static assets at edge locations

Caches:
- Images
- CSS
- JavaScript
- Restaurant thumbnails

TTL:
- 24 hours for static assets

Benefits:
- Removes image traffic from Node.js servers
- Reduces NIC saturation
- Improves latency

---

## Application Load Balancer (ALB)

Purpose:
- Distribute traffic across multiple Node.js instances

Features:
- SSL termination
- Health checks
- Traffic routing
- Auto-scaling integration

Benefits:
- Removes single server bottleneck
- Prevents overload on one instance

---

## Multiple Node.js Instances

Purpose:
- Horizontally scale API layer

Auto-Scaling Trigger:
- CPU > 70%
- Request count spike
- ALB latency increase

Benefits:
- Handles traffic spikes dynamically
- Improves availability

---

## Redis Cache

Purpose:
- Reduce database reads

Caches:
- Restaurant menus
- Promo validation
- Homepage feeds
- Frequently accessed data

TTL:
- 5–15 minutes depending on endpoint

Benefits:
- Reduces PostgreSQL load
- Prevents promo race conditions
- Improves response time

---

## PgBouncer

Purpose:
- PostgreSQL connection pooling

Benefits:
- Multiplexes thousands of app connections
- Prevents DB connection exhaustion
- Reduces idle DB sessions

---

## PostgreSQL Primary + Read Replicas

Primary DB Handles:
- Writes
- Orders
- Payments
- Transactions

Read Replica Handles:
- Restaurant browsing
- Menus
- Search queries

Benefits:
- Separates read and write workloads
- Improves scalability

---

## SQS Payment Queue

Purpose:
- Make payment processing asynchronous

Flow:
1. User places order
2. Payment request enters queue
3. Worker processes payment
4. Order updated after payment confirmation

Benefits:
- Removes synchronous payment bottleneck
- Prevents DB connections from being held during API wait

---

## Payment Worker Service

Purpose:
- Consume SQS messages
- Call Razorpay API
- Retry failed payments safely

Benefits:
- Isolates payment latency
- Improves reliability

---

# Section 4 — Component Justification Table

| Component | Failure Prevented | How It Prevents Failure |
|------------|------------------|--------------------------|
| CloudFront CDN | Static asset NIC saturation | Static assets served from edge instead of Node.js |
| ALB | Single server overload | Distributes traffic across instances |
| Multiple Node.js Instances | Event loop saturation | Horizontal scaling increases request capacity |
| Redis Cache | Database overload | Reduces DB reads significantly |
| Redis Locks | Promo race condition | Atomic operations prevent duplicate coupon usage |
| PgBouncer | PostgreSQL connection exhaustion | Pools and multiplexes DB connections |
| Read Replicas | Primary DB overload | Read traffic isolated from write traffic |
| SQS Queue | Payment amplification | Async queue removes blocking payment calls |
| Payment Workers | Razorpay latency impact | Dedicated workers isolate payment processing |

---

# Section 5 — Expected Scalability Improvements

| Layer | Old Capacity | New Capacity |
|-------|---------------|---------------|
| Node.js API | ~15k RPS | 300k+ RPS horizontally scalable |
| PostgreSQL | 100 connections | Thousands via PgBouncer |
| Static Assets | Single NIC | Global CDN edge delivery |
| Payments | Synchronous | Asynchronous queued processing |

---

# Section 6 — Final Outcome

The redesigned architecture removes all major single points of failure present in the original monolith.

Key improvements:
- Horizontal scalability
- Distributed caching
- Asynchronous payments
- Read/write DB separation
- Edge CDN delivery
- Auto-scaling compute layer

The system is now capable of surviving major traffic spikes such as World Cup Final promotional events.