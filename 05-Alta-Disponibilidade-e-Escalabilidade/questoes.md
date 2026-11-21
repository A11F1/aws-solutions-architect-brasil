# Questions — High Availability and Scalability

## Question 1
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

A company has a REST API using EC2 with Auto Scaling. Corporate customers need to add the API's IPs to their firewall allowlist. Which type of load balancer solves this with a fixed IP?

- A) ALB with static IP
- B) NLB with Elastic IP per AZ
- C) CLB with static IP
- D) ALB with wildcard SSL

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

NLB allows associating one Elastic IP per Availability Zone, providing static and predictable IP addresses. ALB uses DNS and the IP can change. CLB is legacy and has no reliable static IP. Wildcard SSL does not solve the IP problem.

**Key concept:** NLB with Elastic IP for client-side IP allowlisting
</details>

## Question 2
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

A multi-tenant SaaS platform routes traffic to different target groups based on the request hostname: `tenant-a.app.com` goes to TG-A and `tenant-b.app.com` goes to TG-B. Which ALB feature implements this?

- A) Path-based routing
- B) Host-based routing
- C) IP-based routing
- D) Weighted target groups

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Host-based routing uses the HTTP Host header to select the target group according to the request hostname. Path-based uses the URL path (/api, /admin). IP-based routing does not exist as a listener rule on the ALB. Weighted target groups split traffic proportionally between groups, not by hostname.

**Key concept:** host-based routing on the ALB for multi-tenant by hostname
</details>

## Question 3
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

A team wants to use an ALB for a canary release, sending 5% of traffic to the new version and 95% to the current version. Which ALB feature allows this?

- A) Host-based routing with two listeners
- B) Weighted target groups in a single listener rule
- C) NLB with cross-zone balancing
- D) Auto Scaling Group with a weighted Launch Template

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

ALB supports weighted target groups in a listener rule, distributing traffic proportionally between two or more target groups in the same action. It is the standard mechanism for canary and gradual blue/green. Host-based routing routes by hostname, not by weight. NLB does not have this feature. The ASG does not control traffic proportion at the load balancer level.

**Key concept:** weighted target groups on the ALB for canary release
</details>

## Question 4
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

A company wants to inspect all inbound internet traffic before it reaches the application, using a third-party next-gen firewall running on EC2. Which type of load balancer supports this appliance insertion model transparently?

- A) ALB with Lambda authorizer
- B) NLB with TLS pass-through
- C) GWLB with VPC routing to appliances in the target group
- D) CLB with proxy protocol

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

GWLB operates at layer 3 with the GENEVE protocol, allowing transparent insertion of security appliances into the traffic path. Packets are sent to the GWLB endpoint, processed by the appliances in the target group and returned to the GWLB, which redirects them to the destination. ALB, NLB and CLB do not support this IP packet encapsulation model.

**Key concept:** GWLB for transparent insertion of security appliances
</details>

## Question 5
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

An ASG is configured with EC2-only health check. An instance is running but the application process has hung and returns HTTP 500 for any request. Will the ASG replace it automatically?

- A) Yes, the EC2 health check detects any application failure
- B) No, EC2 health check validates only the instance status, not the application
- C) Yes, the ALB notifies the ASG automatically with no additional configuration
- D) No, it is necessary to restart the instance manually

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The EC2 health check only detects hardware/OS status failures (instance status check and system status check). A hung application returning HTTP 500 is still in the "running" state as far as EC2 is concerned. For the ASG to replace instances with a failing application, you must enable the ELB health check on the ASG, which evaluates health in the ALB's target group.

**Key concept:** ELB health check on the ASG to detect application failures
</details>

## Question 6
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

An application receives predictable peaks every Monday at 8 AM when employees arrive at work. The team wants to ensure the instances are already available before the peak, avoiding scale-out latency. Which scaling policy is most suitable?

- A) Target Tracking with CPUUtilization
- B) Scheduled Scaling increasing capacity at 7:45 AM every Monday
- C) Step Scaling with an alarm on CPU > 70%
- D) Manual scaling every Monday morning

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Scheduled Scaling brings the capacity increase ahead of the expected peak, avoiding the lag inherent to reactive policies. Target Tracking and Step Scaling are reactive: they wait for the metric to rise before acting — arriving late for abrupt peaks. Manual scaling depends on human intervention. For predictable peaks, Scheduled (or Predictive with history) is the answer.

**Key concept:** Scheduled Scaling to provision before predictable peaks
</details>

## Question 7
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

A company wants to ensure that when new EC2 instances are added via the ASG, the monitoring agent is installed and configured before the instance starts receiving traffic from the load balancer. Which mechanism should be used?

- A) User Data with an installation script
- B) Lifecycle Hook on EC2_INSTANCE_LAUNCHING with a Lambda that installs the agent
- C) Golden AMI with the agent pre-installed
- D) AWS Config rule to verify the agent

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

A Lifecycle Hook in `Pending:Wait` keeps the instance out of the ELB target group while the automation runs. A Lambda (triggered via SNS/SQS/EventBridge) installs and configures the agent and then calls `CompleteLifecycleAction(CONTINUE)`. User Data runs at boot but does not guarantee that the instance only joins the load balancer afterwards — the hook does. A Golden AMI is good for a pre-installed agent, but not for dynamic configuration.

**Key concept:** Lifecycle Hook Pending:Wait for custom initialization before the ELB
</details>

## Question 8
**Domain:** Design Cost-Optimized Architectures  
**Difficulty:** Medium

An ALB has two target groups in two AZs: TG-A with 10 instances in AZ1 and TG-B with 2 instances in AZ2. With cross-zone load balancing enabled on the ALB, how is traffic distributed?

- A) 50% to AZ1 and 50% to AZ2
- B) Evenly across the 12 instances (~8.3% each)
- C) 83% to TG-A and 17% to TG-B
- D) 100% to TG-A because it has more instances

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

With cross-zone load balancing enabled, each ALB node distributes traffic equally across all healthy instances in all AZs, regardless of how many are in each AZ. 12 instances = ~8.3% each. Without cross-zone, each node distributes only to instances in its own AZ (50% to each AZ, so AZ1 with 10 instances gives each one 5% and AZ2 with 2 instances gives each one 25%).

**Key concept:** cross-zone load balancing distributes evenly across all instances
</details>

## Question 9
**Domain:** Design Resilient Architectures  
**Difficulty:** Hard

An ASG is performing scale-in. The termination policy is the default. AZ1 has 4 instances and AZ2 has 2. Which instance is selected for termination first?

- A) The newest instance in any AZ
- B) An instance from AZ2, since it has fewer and is unbalanced
- C) An instance from AZ1, as it is the AZ with the most instances, followed by the oldest by launch template
- D) Random among all instances

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

The ASG's default termination policy: (1) selects the AZ with the most instances to rebalance; (2) within that AZ, selects the instance with the oldest launch template or configuration; (3) if tied, the one closest to the next chargeable hour. AZ1 with 4 instances is selected first. The oldest by template would be terminated.

**Key concept:** default termination policy: AZ with the most instances → oldest by template
</details>

## Question 10
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

A service needs access to an internal API running in a VPC in another account, without exposing traffic to the internet and without creating a VPC Peering. Which architecture meets this requirement?

- A) Internet Gateway with a restricted security group
- B) NLB with VPC Endpoint Service (PrivateLink) in the producer account
- C) Multi-Region ALB with CloudFront
- D) Site-to-site VPN between the accounts

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

AWS PrivateLink uses an NLB as the backend of a VPC Endpoint Service. The consumer creates an Interface Endpoint (VPC Endpoint) that appears as an ENI in its VPC with a private IP. Traffic never leaves the AWS network and requires no peering, routing, or internet exposure. VPN is more complex and does not solve the problem elegantly.

**Key concept:** NLB + PrivateLink for private cross-account service exposure without peering
</details>

## Question 11
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

An EC2 instance in the ASG is processing a long task when scale-in is triggered. Which mechanism ensures the task completes before the instance is terminated?

- A) Cooldown period of 3600 seconds
- B) Connection Draining on the ELB
- C) Lifecycle Hook on EC2_INSTANCE_TERMINATING
- D) Suspended scaling for scale-in

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

A Lifecycle Hook in `Terminating:Wait` keeps the instance in a waiting state before the actual termination. The automation can monitor task completion and call `CompleteLifecycleAction(CONTINUE)` when finished. Connection Draining only drains active HTTP connections from the ELB, it does not wait for background tasks. Cooldown affects when the next scaling occurs, not the timing of one already started. Suspend would interrupt scaling.

**Key concept:** Lifecycle Hook Terminating:Wait for long-running tasks on scale-in
</details>

## Question 12
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

An architect needs to create a fleet of EC2 instances using multiple types (m5.large, m5.xlarge, m6i.large) with a mix of On-Demand and Spot to reduce cost. Which feature allows this?

- A) Launch Configuration with multiple types
- B) Launch Template with type overrides + Mixed Instance Policy on the ASG
- C) Multiple separate ASGs per type
- D) Dedicated Host with type multiplexing

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Launch Template supports instance type overrides. On the ASG, the Mixed Instance Policy allows defining multiple types and the On-Demand/Spot distribution. The ASG chooses the cheapest available from the pool of defined types. Launch Configuration does not support multiple types. Multiple ASGs increase operational complexity. Dedicated Host has nothing to do with Spot type diversification.

**Key concept:** Launch Template with Mixed Instance Policy for a diversified fleet
</details>

## Question 13
**Domain:** Design Resilient Architectures  
**Difficulty:** Easy

A company wants to ensure that even if an ASG instance looks healthy to EC2, it will be replaced if the application does not return HTTP 200 on health check requests. Which configuration implements this?

- A) Create a CloudWatch alarm for response code
- B) Enable ELB health check on the ASG and configure the health check on the target group
- C) Use enhanced monitoring on EC2
- D) Configure CloudTrail to detect HTTP errors

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

For the ASG to consider application health checks, you need to: (1) configure the health check on the ALB target group with the expected path and success code; (2) enable the ELB health check on the ASG. That way, instances that fail the ALB health check are marked unhealthy in the ASG and replaced. A CloudWatch alarm does not act on the ASG automatically in this way. Enhanced monitoring provides metrics but does not replace instances.

**Key concept:** enable ELB health check on the ASG + configure TG health check
</details>

## Question 14
**Domain:** Design Cost-Optimized Architectures  
**Difficulty:** Medium

A team wants to scale the ASG based on the number of visible messages in the SQS queue (ApproximateNumberOfMessagesVisible). Which scaling policy directly supports custom CloudWatch metrics?

- A) Scheduled Scaling
- B) Predictive Scaling
- C) Target Tracking Scaling with a custom CloudWatch metric
- D) EC2 Auto Recovery

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

Target Tracking supports custom CloudWatch metrics in addition to predefined metrics. You can configure ApproximateNumberOfMessagesVisible/number-of-instances as the target to keep a constant message load per instance. Scheduled is time-based. Predictive uses standard metric history. Auto Recovery is for hardware failures, not scaling.

**Key concept:** Target Tracking with a custom CloudWatch metric to scale by SQS
</details>

## Question 15
**Domain:** Design Resilient Architectures  
**Difficulty:** Hard

A company has an ALB with an HTTPS listener on port 443. The SSL certificate is configured on the listener. The backend application runs HTTP on port 8080. An architect needs to ensure that ALB → EC2 communication is also encrypted. Which configuration implements this?

- A) The ALB cannot establish HTTPS to targets, only HTTP
- B) Configure the target group protocol as HTTPS on port 443 and install a certificate on the instances
- C) Use an NLB with TLS pass-through
- D) Configure an SSL policy on the ALB to automatically encrypt the backend

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The ALB supports HTTPS as a target group protocol. Configure the target group with the HTTPS protocol and install certificates on the instances (self-signed or ACM via Private CA). The ALB establishes HTTPS to each instance. Option A is wrong — the ALB does support HTTPS backends. NLB with TLS pass-through is a different solution. The SSL policy on the listener only controls the TLS negotiation from the client to the ALB.

**Key concept:** HTTPS target group for end-to-end encryption on the ALB
</details>
