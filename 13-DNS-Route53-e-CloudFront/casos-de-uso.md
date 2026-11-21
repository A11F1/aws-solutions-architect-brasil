# Use Cases — Module 08: DNS, Route 53, and CloudFront

## Case 1: Automatic Failover with Route 53 Health Checks

**Scenario:** A critical application in us-east-1 (primary) with automatic failover to eu-west-1 in case of a regional failure.

**Architecture:**
```
Route 53 (api.company.com)
  ├── Record PRIMARY: ALB us-east-1 [FAILOVER + active Health Check]
  │     └── Health Check: HTTP /health, every 30s, threshold of 3 failures
  └── Record SECONDARY: ALB eu-west-1 [FAILOVER, HC not required]

Failover Flow:
1. Health Checker detects failure (3 consecutive failing checks)
2. Route 53 marks the PRIMARY record as UNHEALTHY
3. DNS starts answering with the SECONDARY
4. Low TTL (60s) = fast propagation
```

**Health Check configuration for a private resource:**
- CloudWatch Metric Alarm (based on ALB HealthyHostCount = 0)
- Route 53 HC of type "CloudWatch Alarm" referencing the alarm
- No need to expose a public endpoint for the health check

---

## Case 2: CloudFront with Lambda@Edge for JWT Authentication

**Scenario:** A RESTful API served via CloudFront + API Gateway. Each request must have a valid JWT verified at the edge, without overloading API Gateway.

**Architecture:**
```
Client → CloudFront → Lambda@Edge (Viewer Request) → API Gateway → Lambda
                            │
                  [Validates JWT signed with RS256]
                  [If invalid → 401 Unauthorized]
                  [If valid → adds user-id header → origin]

Lambda@Edge (us-east-1, replicated globally):
- Node.js: verifies the JWT signature with a public key stored in CloudFront KVS
- < 5ms overhead
- No backend call to validate the token → scales indefinitely
```

**Cache Policy:**
- `Authorization` header in the cache key → each unique token = cache miss (do not cache user data)
- For public content: no Authorization in the cache key → maximum cache hit

---

## Case 3: Global Performance with Route 53 + CloudFront

**Scenario:** A global SaaS application serves static content, a dynamic API, and video assets to users on 6 continents.

**Layered Architecture:**
```
DNS: Route 53
  └── app.company.com → Alias → CloudFront Distribution

CloudFront Distribution:
  ├── Behavior: /static/* → S3 Bucket (TTL: 1 year)
  │     └── Cache-Control: max-age=31536000, immutable
  ├── Behavior: /api/* → ALB us-east-1 (TTL: 0, no-cache)
  │     └── Origin Request Policy: forward all headers + cookies
  └── Behavior: /videos/* → S3 Bucket (Signed URLs, TTL: 1h)

Route 53 Latency Routing to the ALB:
  ├── ALB us-east-1 (Americas latency)
  ├── ALB eu-west-1 (Europe latency)
  └── ALB ap-southeast-1 (Asia latency)
```

**Result:** Static content is served from the nearest CloudFront edge (< 20ms). The dynamic API goes to the regional ALB with the lowest latency. Videos: time-limited access controlled with Signed URLs.

---

## Case 4: DNS Migration with Weighted Routing

**Scenario:** A company migrates an application from on-premises to AWS with zero downtime. It needs to validate the new infrastructure with real traffic gradually.

**Migration Strategy (Canary Deployment):**
```
app.company.com (public hosted zone)
  ├── A record: 203.0.113.10 (on-prem) — Weight: 100
  └── A record: ALB AWS        — Weight: 0 (starts at 0 = no traffic)

Week 1: on-prem 95 / AWS 5  → validate AWS logs
Week 2: on-prem 80 / AWS 20 → monitor performance
Week 3: on-prem 50 / AWS 50 → verify DB migration
Week 4: on-prem 0  / AWS 100 → migration complete
```

**Monitoring during migration:**
- CloudWatch Dashboards for both environments in parallel
- Route 53 Health Check on both endpoints
- Low TTL (60s) for agility in weight adjustments

---

## Case 5: DDoS Protection with WAF + Shield Advanced + CloudFront

**Scenario:** An e-commerce platform suffers volumetric attacks and credential stuffing attempts (login brute force).

**Protection Layers:**
```
Internet
  └── CloudFront + Shield Advanced (automatic L3/L4 DDoS)
        └── AWS WAF (L7):
              ├── AWSManagedRulesCommonRuleSet → OWASP Top 10
              ├── AWSManagedRulesBotControlRuleSet → Bots
              ├── Rate-based rule: > 1000 req/5min per IP → BLOCK
              │   (prevents credential stuffing on /login)
              └── IP Set rule: blocklist of known malicious IPs

              After WAF → ALB → App Servers
```

**Attack response:**
- DRT (DDoS Response Team) from Shield Advanced accesses your account for manual mitigation
- WAF Logs for analysis + EventBridge rule for automatic alerts via SNS
- AWS credits for extra scaling costs caused by the DDoS (Shield Advanced benefit)
