# Hands-On Lab — Migration and Transfer: DMS + S3 Transfer (Module 15)

> **Region:** us-east-1 | **Estimated cost:** ~$0.50 (DMS t3.micro replication instance, < 1 hour of use)  
> **Prerequisites:** AWS CLI configured, Terraform >= 1.5, MySQL client

---

## Objective

Simulate an on-premises MySQL database migration to RDS using DMS, and practice transferring data between S3 buckets (simulating a storage migration).

---

## Part 1 — Base Infrastructure with Terraform

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

provider "aws" {
  region = "us-east-1"
}

data "aws_caller_identity" "current" {}

locals {
  prefix     = "lab-migracao"
  account_id = data.aws_caller_identity.current.account_id
}

# VPC for the lab
resource "aws_vpc" "lab" {
  cidr_block           = "10.10.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  tags = { Name = "${local.prefix}-vpc" }
}

resource "aws_internet_gateway" "lab" {
  vpc_id = aws_vpc.lab.id
  tags   = { Name = "${local.prefix}-igw" }
}

# Public subnet (for access to the source MySQL)
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.lab.id
  cidr_block              = "10.10.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = { Name = "${local.prefix}-public-a" }
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.lab.id
  cidr_block              = "10.10.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = { Name = "${local.prefix}-public-b" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.lab.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.lab.id
  }
}

resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public.id
}

# Security Group for RDS
resource "aws_security_group" "rds" {
  name   = "${local.prefix}-rds-sg"
  vpc_id = aws_vpc.lab.id

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["10.10.0.0/16"]
    description = "MySQL inside the VPC"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# RDS MySQL — TARGET database
resource "aws_db_subnet_group" "lab" {
  name       = "${local.prefix}-subnet-group"
  subnet_ids = [aws_subnet.public_a.id, aws_subnet.public_b.id]
}

resource "aws_db_instance" "destino" {
  identifier        = "${local.prefix}-destino"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  storage_type      = "gp2"

  db_name  = "loja_destino"
  username = "admin"
  password = "Lab@12345!"  # Use Secrets Manager in production

  db_subnet_group_name   = aws_db_subnet_group.lab.name
  vpc_security_group_ids = [aws_security_group.rds.id]

  skip_final_snapshot = true
  publicly_accessible = false

  tags = { Environment = "lab", Module = "15-migracao" }
}

# IAM Role for DMS
resource "aws_iam_role" "dms" {
  name = "${local.prefix}-dms-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "dms.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "dms_vpc" {
  role       = aws_iam_role.dms.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonDMSVPCManagementRole"
}

# DMS Subnet Group
resource "aws_dms_replication_subnet_group" "lab" {
  replication_subnet_group_description = "Lab DMS subnet group"
  replication_subnet_group_id          = "${local.prefix}-dms-subnet"
  subnet_ids                           = [aws_subnet.public_a.id, aws_subnet.public_b.id]
}

# DMS Replication Instance
resource "aws_dms_replication_instance" "lab" {
  replication_instance_class  = "dms.t3.micro"
  replication_instance_id     = "${local.prefix}-replication"
  replication_subnet_group_id = aws_dms_replication_subnet_group.lab.id
  vpc_security_group_ids      = [aws_security_group.rds.id]
  publicly_accessible         = false
  allocated_storage           = 20
}

output "rds_endpoint" {
  value = aws_db_instance.destino.endpoint
}

output "dms_instance_arn" {
  value = aws_dms_replication_instance.lab.replication_instance_arn
}
```

---

## Part 2 — Source MySQL Database (simulated locally)

```bash
# Option A: Use Docker to simulate an on-premises database
docker run -d \
  --name mysql-origem \
  -e MYSQL_ROOT_PASSWORD=Lab@12345! \
  -e MYSQL_DATABASE=loja_origem \
  -p 3306:3306 \
  mysql:8.0

sleep 20  # wait for MySQL to initialize

# Populate the source database with test data
docker exec -i mysql-origem mysql -uroot -pLab@12345! loja_origem << 'SQL'
CREATE TABLE clientes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL,
  email VARCHAR(150) UNIQUE NOT NULL,
  cpf CHAR(14) NOT NULL,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE produtos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(200) NOT NULL,
  preco DECIMAL(10,2) NOT NULL,
  estoque INT DEFAULT 0
);

CREATE TABLE pedidos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id INT REFERENCES clientes(id),
  produto_id INT REFERENCES produtos(id),
  quantidade INT NOT NULL,
  total DECIMAL(10,2),
  status ENUM('pendente','aprovado','enviado','entregue') DEFAULT 'pendente',
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Sample data
INSERT INTO clientes (nome, email, cpf) VALUES
  ('João Silva', 'joao@exemplo.com', '000.000.000-00'),
  ('Maria Santos', 'maria@exemplo.com', '111.111.111-11'),
  ('Carlos Oliveira', 'carlos@exemplo.com', '222.222.222-22');

INSERT INTO produtos (nome, preco, estoque) VALUES
  ('Notebook Pro', 2500.00, 10),
  ('Mouse Wireless', 89.90, 50),
  ('Monitor 27"', 1200.00, 5);

INSERT INTO pedidos (cliente_id, produto_id, quantidade, total, status) VALUES
  (1, 1, 1, 2500.00, 'aprovado'),
  (2, 2, 3, 269.70, 'entregue'),
  (3, 3, 2, 2400.00, 'pendente'),
  (1, 2, 1, 89.90, 'enviado');

SELECT 'Data inserted successfully' AS status;
SQL
```

---

## Part 3 — Configure DMS

```bash
# Deploy infrastructure
terraform init && terraform apply -auto-approve

RDS_ENDPOINT=$(terraform output -raw rds_endpoint)
DMS_ARN=$(terraform output -raw dms_instance_arn)

echo "RDS Target: $RDS_ENDPOINT"
echo "DMS Instance: $DMS_ARN"

# Create SOURCE endpoint (local MySQL via the machine's public IP)
MY_IP=$(curl -s ifconfig.me)

SOURCE_ARN=$(aws dms create-endpoint \
  --endpoint-identifier "lab-source-mysql" \
  --endpoint-type source \
  --engine-name mysql \
  --server-name "$MY_IP" \
  --port 3306 \
  --database-name loja_origem \
  --username root \
  --password "Lab@12345!" \
  --query 'Endpoint.EndpointArn' --output text)

# Create TARGET endpoint (RDS)
TARGET_ARN=$(aws dms create-endpoint \
  --endpoint-identifier "lab-target-rds" \
  --endpoint-type target \
  --engine-name mysql \
  --server-name "${RDS_ENDPOINT%:*}" \
  --port 3306 \
  --database-name loja_destino \
  --username admin \
  --password "Lab@12345!" \
  --query 'Endpoint.EndpointArn' --output text)

# Test connection to the endpoints
aws dms test-connection \
  --replication-instance-arn "$DMS_ARN" \
  --endpoint-arn "$SOURCE_ARN"

aws dms test-connection \
  --replication-instance-arn "$DMS_ARN" \
  --endpoint-arn "$TARGET_ARN"

# Create Full Load task
TASK_ARN=$(aws dms create-replication-task \
  --replication-task-identifier "lab-full-load" \
  --source-endpoint-arn "$SOURCE_ARN" \
  --target-endpoint-arn "$TARGET_ARN" \
  --replication-instance-arn "$DMS_ARN" \
  --migration-type full-load \
  --table-mappings '{
    "rules": [{
      "rule-type": "selection",
      "rule-id": "1",
      "rule-name": "1",
      "object-locator": {
        "schema-name": "loja_origem",
        "table-name": "%"
      },
      "rule-action": "include"
    }]
  }' \
  --query 'ReplicationTask.ReplicationTaskArn' --output text)

# Start the task
aws dms start-replication-task \
  --replication-task-arn "$TASK_ARN" \
  --start-replication-task-type start-replication

# Monitor progress
watch -n 10 'aws dms describe-replication-tasks \
  --filters Name=replication-task-arn,Values='"$TASK_ARN"' \
  --query "ReplicationTasks[0].[Status, ReplicationTaskStats.TablesLoaded, ReplicationTaskStats.TablesLoading]" \
  --output table'
```

---

## Part 4 — Validate the Migration and S3 Transfer

```bash
# Verify data in the target RDS
mysql -h "${RDS_ENDPOINT%:*}" -u admin -p"Lab@12345!" loja_destino << 'SQL'
SELECT 'Clientes:' AS tabela, COUNT(*) AS registros FROM clientes
UNION ALL
SELECT 'Produtos:', COUNT(*) FROM produtos
UNION ALL  
SELECT 'Pedidos:', COUNT(*) FROM pedidos;
SQL

# Simulate S3 data migration (Cross-Region or Cross-Account)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
SOURCE_BUCKET="lab-migracao-source-${ACCOUNT_ID}"
DEST_BUCKET="lab-migracao-dest-${ACCOUNT_ID}"

# Create buckets
aws s3api create-bucket --bucket "$SOURCE_BUCKET"
aws s3api create-bucket --bucket "$DEST_BUCKET"

# Create test data
for i in $(seq 1 20); do
  echo "Data file ${i}: $(date) - migration data" > /tmp/arquivo_${i}.txt
  aws s3 cp /tmp/arquivo_${i}.txt "s3://${SOURCE_BUCKET}/dados/arquivo_${i}.txt"
done

echo "20 files created in the source bucket."

# Migrate using S3 Sync
aws s3 sync "s3://${SOURCE_BUCKET}/" "s3://${DEST_BUCKET}/" \
  --metadata-directive COPY \
  --sse aws:kms

# Verify integrity
SOURCE_COUNT=$(aws s3 ls "s3://${SOURCE_BUCKET}/dados/" --recursive | wc -l)
DEST_COUNT=$(aws s3 ls "s3://${DEST_BUCKET}/dados/" --recursive | wc -l)
echo "Source files: $SOURCE_COUNT | Target files: $DEST_COUNT"

# Enable S3 replication (CRR - Cross-Region Replication)
# Requires a bucket in a different region
echo "For CRR: create a bucket in us-west-2 and configure a replication rule"
```

---

## Cleanup

```bash
# Stop and delete DMS
aws dms stop-replication-task --replication-task-arn "$TASK_ARN"
sleep 30
aws dms delete-replication-task --replication-task-arn "$TASK_ARN"
aws dms delete-endpoint --endpoint-arn "$SOURCE_ARN"
aws dms delete-endpoint --endpoint-arn "$TARGET_ARN"

# Destroy Terraform (RDS, DMS instance, VPC)
terraform destroy -auto-approve

# Clean up S3
aws s3 rm "s3://${SOURCE_BUCKET}" --recursive
aws s3 rm "s3://${DEST_BUCKET}" --recursive
aws s3api delete-bucket --bucket "$SOURCE_BUCKET"
aws s3api delete-bucket --bucket "$DEST_BUCKET"

# Stop Docker
docker stop mysql-origem && docker rm mysql-origem

# Clean up temporary files
rm -f /tmp/arquivo_*.txt
```

---

## What You Learned

- DMS requires: replication instance + source endpoint + target endpoint + task
- Full Load migration type: complete snapshot, no continuous replication
- Full Load + CDC type: initial Full Load + real-time change capture
- DMS supports homogeneous and heterogeneous migrations (SCT required for heterogeneous)
- `aws s3 sync` is idempotent — it only transfers new or modified files
- The replication instance is billed hourly even with no tasks running — destroy it after use
