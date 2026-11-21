# Hands-On Lab — RDS: Multi-AZ, Read Replica, and Backup (Module 06)

> **Region:** us-east-1 | **Estimated cost:** ~$0.30 (RDS db.t3.micro, < 1h of use)  
> **Prerequisites:** AWS CLI configured, Terraform >= 1.5, MySQL client

---

## Objective

Create an RDS MySQL instance with Multi-AZ, promote a Read Replica, take a manual backup, and monitor metrics via CloudWatch.

---

## Part 1 — Terraform: RDS Multi-AZ + Read Replica

```hcl
# main.tf
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" { region = "us-east-1" }

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# Subnet Group using the default VPC
resource "aws_db_subnet_group" "lab" {
  name       = "lab-rds-subnet-group"
  subnet_ids = data.aws_subnets.default.ids
}

# Security Group for RDS
resource "aws_security_group" "rds" {
  name   = "lab-rds-sg"
  vpc_id = data.aws_vpc.default.id

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # In prod: specific IP or app SG
    description = "MySQL lab"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Custom Parameter Group
resource "aws_db_parameter_group" "lab" {
  name   = "lab-mysql8-pg"
  family = "mysql8.0"

  parameter {
    name  = "max_connections"
    value = "100"
  }

  parameter {
    name  = "slow_query_log"
    value = "1"
  }

  parameter {
    name  = "long_query_time"
    value = "2"
  }
}

# RDS MySQL Multi-AZ — primary instance
resource "aws_db_instance" "primary" {
  identifier        = "lab-mysql-primary"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  storage_type      = "gp2"

  db_name  = "loja"
  username = "admin"
  password = "Lab@12345Secure!"

  db_subnet_group_name   = aws_db_subnet_group.lab.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  parameter_group_name   = aws_db_parameter_group.lab.name

  multi_az              = true  # HA: standby in another AZ
  publicly_accessible   = true  # for lab; false in production
  skip_final_snapshot   = true

  backup_retention_period = 7  # 7 days of automated backups
  backup_window           = "03:00-04:00"
  maintenance_window      = "Mon:04:00-Mon:05:00"

  deletion_protection = false  # true in production

  tags = { Environment = "lab", Module = "06-rds" }
}

# Read Replica (same region)
resource "aws_db_instance" "read_replica" {
  identifier_prefix   = "lab-mysql-replica-"
  replicate_source_db = aws_db_instance.primary.identifier
  instance_class      = "db.t3.micro"
  publicly_accessible = true
  skip_final_snapshot = true
  auto_minor_version_upgrade = false

  tags = { Role = "read-replica" }
}

output "primary_endpoint"  { value = aws_db_instance.primary.endpoint }
output "replica_endpoint"  { value = aws_db_instance.read_replica.endpoint }
output "primary_arn"       { value = aws_db_instance.primary.arn }
```

---

## Part 2 — Populate and Test

```bash
terraform init && terraform apply -auto-approve

PRIMARY=$(terraform output -raw primary_endpoint)
REPLICA=$(terraform output -raw replica_endpoint)
RDS_ARN=$(terraform output -raw primary_arn)

echo "Primary: $PRIMARY"
echo "Replica: $REPLICA"

# Populate the primary database
mysql -h "${PRIMARY%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja << 'SQL'
CREATE TABLE produtos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(200) NOT NULL,
  preco DECIMAL(10,2),
  categoria VARCHAR(50),
  estoque INT DEFAULT 0,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_categoria (categoria)
);

INSERT INTO produtos (nome, preco, categoria, estoque) VALUES
  ('Notebook Pro', 3500.00, 'Electronics', 10),
  ('Mouse Wireless', 89.90, 'Peripherals', 50),
  ('Mechanical Keyboard', 350.00, 'Peripherals', 25),
  ('27-inch Monitor', 1200.00, 'Electronics', 8),
  ('Headset USB', 180.00, 'Peripherals', 30);

SELECT COUNT(*) AS total, categoria FROM produtos GROUP BY categoria;
SQL

# Wait for replication (< 1 min in the same region)
sleep 15

# Check on the Read Replica
echo "Checking data on the Read Replica..."
mysql -h "${REPLICA%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja \
  -e "SELECT id, nome, preco FROM produtos ORDER BY id;"

# Try to WRITE to the replica (should fail)
echo "Testing write on the replica (should error)..."
mysql -h "${REPLICA%:*}" -P 3306 -u admin -p"Lab@12345Secure!" loja \
  -e "INSERT INTO produtos VALUES (NULL, 'Test', 1, 'X', 1, NOW());" 2>&1 | head -3
```

---

## Part 3 — Manual Backup and Monitoring

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
RDS_ARN=$(terraform output -raw primary_arn)

# Create manual snapshot
SNAPSHOT_ID="lab-mysql-snapshot-$(date +%Y%m%d%H%M)"
aws rds create-db-snapshot \
  --db-instance-identifier "lab-mysql-primary" \
  --db-snapshot-identifier "$SNAPSHOT_ID"

# Wait for the snapshot to become available
aws rds wait db-snapshot-available \
  --db-snapshot-identifier "$SNAPSHOT_ID"

echo "Snapshot ready: $SNAPSHOT_ID"

# List all snapshots (automated + manual)
aws rds describe-db-snapshots \
  --db-instance-identifier "lab-mysql-primary" \
  --query 'DBSnapshots[*].[DBSnapshotIdentifier, SnapshotType, Status, AllocatedStorage]' \
  --output table

# Monitor CloudWatch metrics
echo ""
echo "=== RDS CloudWatch Metrics ==="
START=$(date -u -d "1 hour ago" +%FT%TZ 2>/dev/null || date -u -v-1H +%FT%TZ)
END=$(date -u +%FT%TZ)

for METRIC in "CPUUtilization" "FreeStorageSpace" "DatabaseConnections" "ReadLatency" "WriteLatency"; do
  VALUE=$(aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name "$METRIC" \
    --dimensions "Name=DBInstanceIdentifier,Value=lab-mysql-primary" \
    --start-time "$START" --end-time "$END" \
    --period 3600 --statistics Average \
    --query 'Datapoints[0].Average' --output text 2>/dev/null)
  echo "  $METRIC: ${VALUE:-no data}"
done

# Create alarm for high CPU
aws cloudwatch put-metric-alarm \
  --alarm-name "lab-rds-high-cpu" \
  --alarm-description "RDS CPU above 80%" \
  --namespace AWS/RDS \
  --metric-name CPUUtilization \
  --dimensions "Name=DBInstanceIdentifier,Value=lab-mysql-primary" \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

echo "CPU alarm created."
```

---

## Cleanup

```bash
# Delete manual snapshot
aws rds delete-db-snapshot --db-snapshot-identifier "$SNAPSHOT_ID"

# Delete alarm
aws cloudwatch delete-alarms --alarm-names "lab-rds-high-cpu"

# Destroy infrastructure
terraform destroy -auto-approve
```

---

## What You Learned

- **Multi-AZ:** synchronous standby in the same region; automatic failover in ~1-2 min; NOT usable for reads
- **Read Replica:** asynchronous; can be cross-region; can be promoted to a standalone instance; serves reads
- **Manual vs automated snapshot:** manual snapshots do not expire; automated ones follow `backup_retention_period`
- **Restaurar snapshot:** creates a new instance (does not replace the current one) — RDS has no in-place instance restore
- **Parameter Group:** engine settings (slow query log, max connections) — applied with a reboot (static) or immediately (dynamic)
