# Hands-On Lab — Monitoring: CloudWatch + CloudTrail + Config (Module 14)

> **Region:** us-east-1 | **Estimated cost:** < $0.50 (CloudWatch first 10 alarms free; CloudTrail first 90 days of Event History free)  
> **Prerequisites:** AWS CLI configured, Terraform >= 1.5

---

## Objective

Implement a complete observability stack: custom metrics, alarms, CloudTrail with alerts for critical actions, and a Config rule.

---

## Part 1 — CloudWatch: Custom Metrics and Alarms

```bash
# Publish a custom metric (simulate an app that counts orders)
aws cloudwatch put-metric-data \
  --namespace "MyApp/Orders" \
  --metric-data '[
    {
      "MetricName": "OrdersCreated",
      "Dimensions": [{"Name": "Environment", "Value": "production"}],
      "Value": 42,
      "Unit": "Count"
    },
    {
      "MetricName": "ErrorRate",
      "Dimensions": [{"Name": "Environment", "Value": "production"}],
      "Value": 2.5,
      "Unit": "Percent"
    }
  ]'

# Verify that the metric was recorded
aws cloudwatch list-metrics \
  --namespace "MyApp/Orders" \
  --query 'Metrics[*].[MetricName, Dimensions[0].Value]' \
  --output table

# Create an alarm for the error rate
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create an SNS topic for alerts
SNS_ARN=$(aws sns create-topic \
  --name lab-monitoring-alerts \
  --query 'TopicArn' --output text)

# Subscribe an email to the topic (replace with your email)
aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol email \
  --notification-endpoint "your-email@example.com"

echo "Confirm the subscription in your email before continuing."
echo "SNS Topic ARN: $SNS_ARN"

# Create an alarm for error rate > 5%
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-high-error-rate" \
  --alarm-description "Application error rate above 5%" \
  --metric-name "ErrorRate" \
  --namespace "MyApp/Orders" \
  --dimensions Name=Environment,Value=production \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 5.0 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions "$SNS_ARN" \
  --ok-actions "$SNS_ARN" \
  --treat-missing-data notBreaching

# Check the alarm state
aws cloudwatch describe-alarms \
  --alarm-names "lab-high-error-rate" \
  --query 'MetricAlarms[*].[AlarmName, StateValue, Threshold]' \
  --output table

# Simulate ALARM (publish a value above the threshold)
aws cloudwatch put-metric-data \
  --namespace "MyApp/Orders" \
  --metric-data '[
    {
      "MetricName": "ErrorRate",
      "Dimensions": [{"Name": "Environment", "Value": "production"}],
      "Value": 15.0,
      "Unit": "Percent"
    }
  ]'

echo "Wait ~5 minutes and check the alarm state..."
```

---

## Part 2 — CloudWatch Logs: Filters and Queries

```bash
# Create a test Log Group
aws logs create-log-group --log-group-name "/lab/application"
aws logs create-log-stream \
  --log-group-name "/lab/application" \
  --log-stream-name "test-stream"

# Inject simulated logs
TIMESTAMP=$(date +%s%3N)

aws logs put-log-events \
  --log-group-name "/lab/application" \
  --log-stream-name "test-stream" \
  --log-events "[
    {\"timestamp\": ${TIMESTAMP}, \"message\": \"INFO: Order 1001 created successfully\"},
    {\"timestamp\": $((TIMESTAMP + 1000)), \"message\": \"ERROR: Failed to process order 1002 - timeout\"},
    {\"timestamp\": $((TIMESTAMP + 2000)), \"message\": \"INFO: Order 1003 created successfully\"},
    {\"timestamp\": $((TIMESTAMP + 3000)), \"message\": \"ERROR: Connection refused to the database\"},
    {\"timestamp\": $((TIMESTAMP + 4000)), \"message\": \"WARN: Abnormal response time: 2500ms\"}
  ]"

# CloudWatch Logs Insights Query
aws logs start-query \
  --log-group-name "/lab/application" \
  --start-time $(($(date +%s) - 3600)) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message
    | filter @message like /ERROR/
    | sort @timestamp desc
    | limit 20'

# Get the result (use the queryId returned above)
# aws logs get-query-results --query-id "QUERY_ID_HERE"

# Create a Metric Filter (counts ERRORs per minute)
aws logs put-metric-filter \
  --log-group-name "/lab/application" \
  --filter-name "CountErrors" \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=MyApp/Logs,metricValue=1,defaultValue=0
```

---

## Part 3 — CloudTrail: Auditing with Alerts

```bash
# Create a bucket for CloudTrail
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
CT_BUCKET="lab-cloudtrail-${ACCOUNT_ID}"

aws s3api create-bucket --bucket "$CT_BUCKET" --region us-east-1

# Create a bucket policy (CloudTrail needs permission to write)
cat > cloudtrail_bucket_policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::${CT_BUCKET}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {"Service": "cloudtrail.amazonaws.com"},
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::${CT_BUCKET}/AWSLogs/${ACCOUNT_ID}/*",
      "Condition": {
        "StringEquals": {"s3:x-amz-acl": "bucket-owner-full-control"}
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket "$CT_BUCKET" \
  --policy file://cloudtrail_bucket_policy.json

# Create the trail
aws cloudtrail create-trail \
  --name "lab-trail" \
  --s3-bucket-name "$CT_BUCKET" \
  --include-global-service-events \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging --name "lab-trail"

# Check the status
aws cloudtrail get-trail-status --name "lab-trail" \
  --query '[IsLogging, LatestDeliveryTime]'

# Create a Metric Filter in CloudWatch to detect DeleteBucket
aws logs create-log-group --log-group-name "CloudTrailLogs"

aws cloudtrail update-trail \
  --name "lab-trail" \
  --cloud-watch-logs-log-group-arn "arn:aws:logs:us-east-1:${ACCOUNT_ID}:log-group:CloudTrailLogs:*" \
  --cloud-watch-logs-role-arn "arn:aws:iam::${ACCOUNT_ID}:role/CloudTrail_CloudWatchLogs_Role"

# Create a filter for bucket deletion actions
aws logs put-metric-filter \
  --log-group-name "CloudTrailLogs" \
  --filter-name "DeleteBucketFilter" \
  --filter-pattern '{ ($.eventName = DeleteBucket) }' \
  --metric-transformations \
    metricName=DeleteBucketCount,metricNamespace=CloudTrailMetrics,metricValue=1

# Alarm for bucket deletion
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-s3-bucket-deleted" \
  --metric-name "DeleteBucketCount" \
  --namespace "CloudTrailMetrics" \
  --statistic Sum \
  --period 300 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions "$SNS_ARN" \
  --treat-missing-data notBreaching
```

---

## Part 4 — AWS Config: Compliance Rule

```bash
# View Config rules already available
aws configservice describe-config-rules \
  --query 'ConfigRules[*].ConfigRuleName' \
  --output table

# Create a rule: EC2 instances without a public IP
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "lab-ec2-no-public-ip",
    "Description": "Checks whether EC2 instances have a public IP associated",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "EC2_INSTANCE_NO_PUBLIC_IP"
    },
    "Scope": {
      "ComplianceResourceTypes": ["AWS::EC2::Instance"]
    }
  }'

# View compliance status (may take a few minutes)
aws configservice describe-compliance-by-config-rule \
  --config-rule-names "lab-ec2-no-public-ip" \
  --query 'ComplianceByConfigRules[*].[ConfigRuleName,Compliance.ComplianceType]' \
  --output table

# View non-compliant resources
aws configservice get-compliance-details-by-config-rule \
  --config-rule-name "lab-ec2-no-public-ip" \
  --compliance-types "NON_COMPLIANT" \
  --query 'EvaluationResults[*].EvaluationResultIdentifier.EvaluationResultQualifier.[ResourceType,ResourceId]' \
  --output table
```

---

## Cleanup

```bash
# Stop CloudTrail
aws cloudtrail stop-logging --name "lab-trail"
aws cloudtrail delete-trail --name "lab-trail"

# Remove the CloudTrail bucket
aws s3 rm "s3://${CT_BUCKET}" --recursive
aws s3api delete-bucket --bucket "$CT_BUCKET"

# Remove alarms
aws cloudwatch delete-alarms \
  --alarm-names "lab-high-error-rate" "lab-s3-bucket-deleted"

# Remove the SNS topic
aws sns delete-topic --topic-arn "$SNS_ARN"

# Remove log groups
aws logs delete-log-group --log-group-name "/lab/application"
aws logs delete-log-group --log-group-name "CloudTrailLogs"

# Remove the Config rule
aws configservice delete-config-rule --config-rule-name "lab-ec2-no-public-ip"

# Remove the temporary file
rm -f cloudtrail_bucket_policy.json
```

---

## What You Learned

- Custom metrics in CloudWatch let you monitor business KPIs (not just infrastructure)
- Alarms can be composed (Composite Alarms) to avoid false positives
- CloudWatch Logs Insights offers SQL-like queries on logs without ETL
- CloudTrail records ALL API calls — essential for auditing
- Metric Filters turn log events into metrics (the bridge between logs and alarms)
- Config evaluates compliance continuously — not only when configured
