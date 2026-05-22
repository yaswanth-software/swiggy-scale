# AWS COST ESTIMATE

# Section 1 — Assumptions

## Traffic Assumptions

- Baseline monthly traffic: Normal SwiftEats operations
- Peak event: India vs Pakistan World Cup Final
- Peak duration: 4 hours
- Peak traffic: ~960,000 RPS burst
- Multi-region CDN delivery enabled

---

# Section 2 — Baseline Infrastructure Cost

## EC2 — Node.js API Servers

Instance Type:
- t3.large

Configuration:
- 6 instances running continuously

Pricing:
- $0.0832/hour per instance

Monthly Cost Formula:

0.0832 × 720 hours × 6

= $359.42/month

---

## RDS PostgreSQL Primary

Instance Type:
- db.r6g.large

Pricing:
- $0.252/hour

Monthly Cost Formula:

0.252 × 720

= $181.44/month

---

## RDS PostgreSQL Read Replicas

Configuration:
- 2 read replicas

Instance Type:
- db.r6g.large

Monthly Cost Formula:

0.252 × 720 × 2

= $362.88/month

---

## ElastiCache Redis Cluster

Configuration:
- 2 cache.r6g.large nodes

Pricing:
- $0.138/hour

Monthly Cost Formula:

0.138 × 720 × 2

= $198.72/month

---

## Application Load Balancer (ALB)

Base Pricing:
- $0.0225/hour

LCU Estimate:
- $0.008 per LCU-hour

Estimated Monthly Cost:

≈ $40/month

---

## CloudFront CDN

Estimated Monthly Transfer:
- 50 TB

Pricing Estimate:
- Approx $0.085/GB

Transfer Cost:

50,000 GB × 0.085

= $4,250/month

Request Cost Estimate:
- $120/month

Total CloudFront Cost:
= $4,370/month

---

## Amazon SQS

Estimated Monthly Messages:
- 120 million payment queue events

Pricing:
- $0.40 per million requests

Monthly Cost Formula:

120 × 0.40

= $48/month

---

# Section 3 — Baseline Monthly Total

| Service | Monthly Cost |
|----------|--------------|
| EC2 API Layer | $359.42 |
| RDS Primary | $181.44 |
| RDS Replicas | $362.88 |
| Redis Cluster | $198.72 |
| ALB | $40 |
| CloudFront | $4,370 |
| SQS | $48 |

## Total Baseline Monthly Cost

= ~$5,560/month

---

# Section 4 — Peak Event Scaling Cost

## Additional EC2 Capacity

Scale-Up During Event:
- Additional 30 t3.large instances

Duration:
- 4 hours

Cost Formula:

30 × 0.0832 × 4

= $9.98

---

## Additional CloudFront Transfer

Estimated Additional Transfer:
- 25 TB

Transfer Cost:

25,000 × 0.085

= $2,125

---

## Additional Redis Scaling

Temporary additional cache nodes:
- 2 extra nodes for 4 hours

Cost Formula:

2 × 0.138 × 4

= $1.10

---

# Section 5 — Peak Event Extra Cost

| Service | Extra Event Cost |
|----------|------------------|
| Additional EC2 | $9.98 |
| Additional CloudFront Transfer | $2,125 |
| Additional Redis Nodes | $1.10 |

## Total Peak Event Cost

≈ $2,136

---

# Section 6 — Business Impact Comparison

## Estimated Outage Cost

Business loss estimate:
- ₹4.2 crore per minute during major event outage

45-minute outage cost:

4.2 × 45

= ₹189 crore loss

---

## Infrastructure ROI

Monthly infrastructure cost:
≈ $5,560/month

Peak event scaling cost:
≈ $2,136

Compared to:
- ₹189 crore potential outage loss

The infrastructure investment is financially justified even if it prevents a single major outage event annually.

The redesigned architecture significantly reduces:
- downtime risk
- revenue loss
- customer churn
- operational instability

---

# Section 7 — Final Recommendation

The baseline architecture cost is relatively small compared to the potential business impact of platform failure during high-profile events.

Recommended production strategy:
- Always-on scalable architecture
- Auto-scaling API tier
- CDN-first static delivery
- Redis caching layer
- Asynchronous payment processing
- Read replica database scaling

The projected infrastructure spend is operationally and financially sustainable for a large-scale food delivery platform.