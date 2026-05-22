# Swiggy Scale Simulation

A large-scale SRE and backend architecture simulation analyzing how a Swiggy-like food delivery platform fails under World Cup Final traffic spikes and how to redesign it for high availability and scalability.

---

# Scenario

SwiftEats is a food delivery platform running on:

- Single Node.js server
- Single PostgreSQL database
- No Redis cache
- No CDN
- No auto-scaling
- Synchronous payment processing

During an India vs Pakistan World Cup Final promotional campaign:

- 180 million users receive push notifications
- 8% click-through rate
- ~14.4 million users open the app
- Traffic concentrated within 60 seconds

The project investigates whether the platform can survive this traffic spike and designs a scalable replacement architecture.

---

# Repository Structure

| Document | Description |
|----------|-------------|
| docs/FAILURE-CASCADE.md | Failure cascade analysis with RPS calculations, bottlenecks, and outage timeline |
| docs/ARCHITECTURE.md | Redesigned scalable distributed architecture with component justifications |
| docs/COST-ESTIMATE.md | AWS infrastructure pricing and peak event cost analysis |
| docs/RUNBOOK.md | Production incident response runbook for high-traffic outages |

---

# Key Findings

## Peak Traffic Calculation

Peak RPS:

(14.4 million users × 4 API calls) / 60 seconds

= ~960,000 requests per second

---

## Major Bottlenecks Identified

- PostgreSQL connection pool exhaustion at 100 connections
- Node.js event loop saturation at ~15k RPS
- Synchronous payment API amplification
- Promo code race conditions
- Static asset NIC saturation due to lack of CDN

---

## Failure Timeline

- T+0s → Notification sent
- T+10s → Database pool saturation begins
- T+20s → Node.js event loop congestion
- T+1m → Platform-wide 500/503 errors
- T+5m → Node.js OOM crash
- T+30m → Incident mitigation starts

---

# Redesigned Architecture Overview

The redesigned architecture replaces the monolith with a distributed, horizontally scalable system using:

- CloudFront CDN
- Application Load Balancer
- Multiple Node.js API instances
- Redis caching layer
- PgBouncer connection pooling
- PostgreSQL primary + read replicas
- SQS asynchronous payment queues
- Dedicated payment worker services

The new system eliminates single points of failure and supports high-scale traffic events safely.

---

# Technologies Covered

- Node.js
- PostgreSQL
- Redis
- AWS EC2
- AWS RDS
- AWS ElastiCache
- AWS CloudFront
- AWS SQS
- Application Load Balancers
- PgBouncer
- Kubernetes / Auto Scaling

---

# Cost Summary

## Baseline Infrastructure Cost

Approximate monthly AWS infrastructure cost:

~$5,560/month

---

## Peak Event Scaling Cost

Additional World Cup Final scaling cost:

~$2,136 for 4-hour event window

---

# Business Impact

Estimated outage loss during peak event:

₹4.2 crore/minute

45-minute outage estimate:

₹189 crore loss

The infrastructure redesign cost is significantly lower than the business impact of a major outage.

---

# Outcome

This project demonstrates:

- Failure cascade analysis
- Capacity planning
- Distributed system redesign
- Cloud infrastructure estimation
- Incident response engineering
- Production reliability thinking

It reflects real-world SRE and platform engineering practices used in high-scale consumer internet systems.