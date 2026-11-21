# Flashcards — Module 08: DNS, Route 53, and CloudFront

---

**Q:** Why can't CNAME be used at the root domain (apex)?
**A:** RFC 1034 forbids CNAME at the zone apex (it cannot coexist with other records such as SOA/NS). AWS created the **Alias record** as a proprietary extension that works at the apex and points to AWS resources at no additional query cost

---

**Q:** What is the default TTL for Route 53 records and what happens during failover?
**A:** TTL is configurable (e.g., 300s = 5 min). During failover, the TTL determines how long DNS resolvers around the world will cache the old address. Low TTL = faster failover, more queries (cost). High TTL = slower failover, fewer queries

---

**Q:** What are the 8 Route 53 routing types?
**A:** Simple, Weighted, Latency, Failover, Geolocation, Geoproximity (Traffic Flow), Multi-Value Answer, IP-based routing

---

**Q:** Which routing policy uses Traffic Flow with a configurable bias?
**A:** **Geoproximity routing** (Traffic Flow). It routes based on geographic distance. The "bias" increases or decreases a resource's region of influence (positive bias → attracts more traffic; negative → repels). Requires Route 53 Traffic Flow

---

**Q:** What is the difference between Geolocation and Latency routing?
**A:** Geolocation: routes based on the client's country/continent (e.g., users from Brazil → BR server). Latency: routes based on MEASURED latency between the client and AWS Regions (e.g., a user in Brazil → server in us-east-1 if it is faster than sa-east-1)

---

**Q:** What is a Calculated Health Check in Route 53?
**A:** It combines the results of multiple individual Health Checks using AND, OR, or NOT. It lets you build logic such as: "consider healthy if at least 2 of 3 endpoints are UP". Useful for being more resilient to false positives

---

**Q:** How do you monitor private resources (no public IP) with Route 53 Health Checks?
**A:** Route 53 Health Checkers operate from the internet. For private resources: create a CloudWatch Metric Alarm based on the resource's metrics → Health Check of type "CloudWatch Alarm". When the Alarm → ALARM: the Health Check fails

---

**Q:** What is the difference between OAI and OAC in CloudFront?
**A:** OAI (Origin Access Identity): legacy, works with standard S3. OAC (Origin Access Control): the new standard, supports all S3 Regions, SSE-KMS, S3 Object Lambda, more HTTP methods. For new projects: always use OAC

---

**Q:** What is a Cache Policy vs an Origin Request Policy in CloudFront?
**A:** Cache Policy: defines what goes into the cache key (headers, cookies, query strings) and the TTL. Origin Request Policy: defines what is sent to the origin (headers, cookies, query strings) IN ADDITION to what is in the cache key. The origin can receive more information than the cache considers

---

**Q:** How much does a CloudFront Cache Invalidation cost?
**A:** First **1,000 paths** of invalidation per month: free. Above that: $0.005 per path. The wildcard `/*` counts as 1 path. Best practice: use asset versioning (hashing in the file name) to avoid invalidations

---

**Q:** What are CloudFront Functions vs Lambda@Edge? Which is faster?
**A:** CloudFront Functions: sub-millisecond, JavaScript, only Viewer Request/Response, cheaper. Lambda@Edge: up to 30s (Origin), Node.js/Python, 4 events including Origin. CF Functions is faster and cheaper; Lambda@Edge is more powerful

---

**Q:** What is AWS Global Accelerator for?
**A:** Improves performance and availability for global applications using 2 static Anycast IPs. Traffic enters through the nearest AWS edge and travels over the AWS private backbone network. Ideal for: fixed IPs (firewall), non-HTTP traffic (generic UDP/TCP), failover in seconds (no DNS TTL)

---

**Q:** What is the difference between Route 53 and Global Accelerator for failover?
**A:** Route 53: DNS-based failover with TTL → can take minutes to propagate. Global Accelerator: failover in seconds (no TTL, Anycast routing), but covers only AWS endpoints. For ultra-fast failover and static IPs: GA. For simplicity and flexibility: Route 53

---

**Q:** What are Route 53 Resolver Inbound vs Outbound Endpoints?
**A:** Inbound: lets on-premises resolve DNS names of AWS resources (queries arrive from outside into the VPC). Outbound: lets resources in the VPC resolve on-premises DNS names (forwarding rules to on-prem DNS servers over DX/VPN)

---

**Q:** Which AWS resources are valid targets of an Alias record in Route 53?
**A:** ALB/NLB/CLB, CloudFront, API Gateway, Elastic Beanstalk, VPC Interface Endpoints, Global Accelerator, S3 website endpoint, other records in the same Route 53 zone. Alias resolves on the DNS server — no extra query cost

---

**Q:** What is a WAF Rate-based rule?
**A:** Limits the number of requests from a single IP within a time window (5 minutes). E.g., block an IP that makes more than 1,000 requests in 5 minutes. Protects against brute force and layer 7 (volumetric) DDoS

---

**Q:** What is AWS Shield Standard vs Advanced?
**A:** Standard: free, automatic for everyone, protects against layer 3/4 DDoS (SYN floods, UDP reflection). Advanced: paid (~$3,000/month), also protects layer 7, 24/7 DRT (DDoS Response Team), cost protection credits for DDoS, detailed reports

---

**Q:** How do you enforce HTTPS on CloudFront?
**A:** On the Behavior: "Viewer Protocol Policy" = "Redirect HTTP to HTTPS" or "HTTPS Only". Use an ACM certificate (us-east-1 Region for CloudFront). For an S3 origin: OAC does not require HTTPS but it is recommended. For custom origins: configure the Origin Protocol Policy

---

**Q:** What is Multi-Value Answer routing in Route 53?
**A:** Returns multiple IPs (up to 8) for the same DNS name, each with a Health Check. Different from Simple (returns all without health checks). Clients generally use the first working IP. It is NOT a load balancer (does not replace an ALB) — it only returns multiple healthy records

---

**Q:** How does CloudFront Geo Restriction work and what is the more granular alternative?
**A:** Geo Restriction: blocks/allows countries via MaxMind GeoIP (returns 403). Applies to the whole distribution or per behavior. More granular alternative: integrate Lambda@Edge that checks the `CloudFront-Viewer-Country` header and applies custom logic (e.g., redirect to a localized version, not just block)
