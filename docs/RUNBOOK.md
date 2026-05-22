# INCIDENT RESPONSE RUNBOOK

# Purpose

This runbook provides operational response steps for handling production incidents during high-traffic events such as World Cup Final promotional campaigns.

Audience:
- Junior engineers
- On-call SREs
- Backend platform engineers

---

# STEP 1 — DETECT

## CloudWatch Alarms

| Metric | Threshold | Severity | Meaning |
|--------|------------|-----------|----------|
| EC2 CPU Utilization | >80% for 5 min | Warning | API compute saturation |
| ALB Target Response Time | >2 seconds | Critical | Backend latency spike |
| HTTP 5xx Error Rate | >5% | Critical | Users receiving server errors |
| PostgreSQL Connections | >90 active connections | Critical | DB pool exhaustion risk |
| Redis Cache Miss Rate | >20% | Warning | Cache degradation |
| SQS Queue Depth | >50,000 messages | Critical | Payment backlog growing |
| RDS CPU Utilization | >75% | Warning | Database saturation |
| CloudFront Origin Errors | >3% | Warning | CDN fallback traffic increasing |

---

# STEP 2 — TRIAGE

## 30-Second Incident Identification Flow

### Step 1 — Check Database Connections

Dashboard:
- CloudWatch → RDS → DatabaseConnections

If:
- Connections > 90
- Query latency rising

Root Cause:
- PostgreSQL connection exhaustion

Action:
- Go to STEP 3A

---

### Step 2 — Check EC2 CPU Usage

Dashboard:
- CloudWatch → EC2 → CPUUtilization

If:
- CPU > 80%
- ALB latency increasing

Root Cause:
- Node.js compute saturation

Action:
- Go to STEP 3B

---

### Step 3 — Check SQS Queue Depth

Dashboard:
- CloudWatch → SQS → ApproximateNumberOfMessagesVisible

If:
- Queue rapidly increasing
- Payment delays reported

Root Cause:
- Payment worker backlog

Action:
- Go to STEP 3C

---

### Step 4 — Check Redis Metrics

Dashboard:
- ElastiCache → CacheHitRate

If:
- Cache hit rate drops below 80%

Root Cause:
- Redis cache degradation

Action:
- Go to STEP 3D

---

# STEP 3 — RESPOND

# STEP 3A — Database Pool Exhaustion

## Symptoms

- Orders timing out
- Checkout failures
- DatabaseConnections > 90

## Immediate Actions

### 1. Check Active Queries

Run:

```bash
SELECT pid, query, state
FROM pg_stat_activity
ORDER BY state;

### 2. Kill Long-Running Queries

```bash
SELECT pg_terminate_backend(pid);
```

for blocked or idle transactions.

### 3. Enable Read Replica Routing

Redirect read-heavy endpoints to replicas.

### 4. Increase PgBouncer Pooling

Adjust:
- max_client_conn
- default_pool_size

## Success Criteria

- DB connections fall below 70
- API latency stabilizes
- Error rate drops below 2%

## Ownership

- Team: Database Platform Team
- Slack: #incident-db

---

# STEP 3B — Compute Saturation

## Symptoms

- EC2 CPU >80%
- High ALB latency
- 502/503 errors

## Immediate Actions

### 1. Increase Auto Scaling Group Size

AWS Console:
- EC2 → Auto Scaling Groups
- Increase desired capacity

OR CLI:

```sql
SELECT pid, query, state
FROM pg_stat_activity
ORDER BY state;
```

### 2. Verify New Instances

Check:
- EC2 instance health
- ALB target registration

## Success Criteria

- CPU utilization <60%
- ALB latency <1 second

## Ownership

- Team: Platform SRE
- Slack: #incident-compute

---

# STEP 3C — Payment Queue Backlog

## Symptoms

- SQS queue depth increasing
- Delayed payment confirmations

## Immediate Actions

### 1. Scale Payment Workers

Increase ECS/Kubernetes worker replicas.

Example:

```bash
kubectl scale deployment payment-worker --replicas=30
```

### 2. Verify Queue Consumption

Check:
- Message dequeue rate
- Worker logs
- Razorpay API latency

## Success Criteria

- Queue depth decreasing continuously
- Payment completion latency <30s

## Ownership

- Team: Payments Team
- Slack: #incident-payments

---

# STEP 3D — Redis Cache Miss Spike

## Symptoms

- Increased DB load
- Cache hit rate below 80%

## Immediate Actions

### 1. Check Redis Memory Usage

Inspect:
- Evictions
- Memory fragmentation

### 2. Scale Redis Cluster

Add read replicas or larger nodes.

### 3. Warm Critical Cache Keys

Preload:
- restaurant feeds
- menu caches
- promo metadata

## Success Criteria

- Cache hit rate >95%
- DB read traffic decreases

## Ownership

- Team: Platform Cache Team
- Slack: #incident-cache

---

# STEP 4 — ROLLBACK

## Rollback Criteria

Rollback only if:
- Error rate remains >10% after mitigation
- New deployment identified as root cause
- Latency continuously increasing

Do NOT rollback if:
- Issue caused by traffic spike only
- Infrastructure scaling resolves issue

---

## Rollback Procedure

### Roll Back Application Version

Example Kubernetes command:

```bash
kubectl rollout undo deployment/swift-api
```

### Verify Rollback Success

Check:
- 5xx error reduction
- Latency normalization
- Stable CPU usage

---

## Critical Warning

NEVER rollback database schema changes during active production incidents unless explicitly approved by the database owner.

Only rollback:
- application code
- API deployments
- worker deployments

---

# STEP 5 — POSTMORTEM

# Postmortem Template

## Incident Title

Example:
- "World Cup Promo Traffic Outage"

---

## Summary

Describe:
- What failed
- User impact
- Duration
- Recovery actions

---

## Timeline

| Time | Event |
|------|------|
| T+0 | Notification sent |
| T+5m | API latency spike |
| T+15m | DB saturation |
| T+30m | Mitigation started |
| T+45m | Recovery stabilized |

---

## Root Cause

Explain:
- Primary technical cause
- Triggering event
- Why safeguards failed

---

## Customer Impact

Include:
- Failed orders
- Revenue impact
- Error rate
- Duration of outage

---

## What Worked Well

Document:
- Successful mitigations
- Monitoring effectiveness
- Team coordination

---

## What Failed

Document:
- Missing alarms
- Slow response
- Capacity gaps
- Operational mistakes

---

## Action Items

| Action Item | Owner | Due Date | Status |
|-------------|-------|-----------|--------|
| Add Redis replica | Platform Team | YYYY-MM-DD | Open |
| Increase DB pooling | DB Team | YYYY-MM-DD | Open |

---

## Lessons Learned

Document:
- Engineering improvements
- Operational improvements
- Monitoring improvements

---

# Final Notes

During high-scale incidents:
- Prioritize restoring service over perfect diagnosis
- Scale horizontally before restarting systems
- Protect the database first
- Communicate updates every 15 minutes

This runbook should be reviewed quarterly and updated after every major production incident.