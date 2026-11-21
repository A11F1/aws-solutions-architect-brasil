# Hands-On Lab — Data and Analytics: Athena + Glue + S3 (Module 12)

> **Region:** us-east-1 | **Estimated cost:** < $1.00 (Athena charges $5/TB scanned; the lab uses ~100 MB)  
> **Prerequisites:** AWS CLI configured, Terraform >= 1.5

---

## Objective

Build a simple analytics pipeline: CSV data in S3 → Glue Crawler catalogs it → Athena queries it — without provisioning any servers.

---

## Part 1 — Infrastructure with Terraform

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
  account_id = data.aws_caller_identity.current.account_id
  prefix     = "lab-analytics"
}

# S3 bucket — raw data
resource "aws_s3_bucket" "raw" {
  bucket        = "${local.prefix}-raw-${local.account_id}"
  force_destroy = true

  tags = { Environment = "lab", Module = "12-analytics" }
}

resource "aws_s3_bucket_versioning" "raw" {
  bucket = aws_s3_bucket.raw.id
  versioning_configuration { status = "Enabled" }
}

# S3 bucket — Athena results
resource "aws_s3_bucket" "athena_results" {
  bucket        = "${local.prefix}-athena-results-${local.account_id}"
  force_destroy = true
}

# Glue Database
resource "aws_glue_catalog_database" "analytics" {
  name = "lab_analytics_db"
}

# IAM Role for the Glue Crawler
resource "aws_iam_role" "glue_role" {
  name = "${local.prefix}-glue-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "glue.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "glue_service" {
  role       = aws_iam_role.glue_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole"
}

resource "aws_iam_role_policy" "glue_s3" {
  name = "${local.prefix}-glue-s3-policy"
  role = aws_iam_role.glue_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject", "s3:PutObject", "s3:ListBucket"]
      Resource = [
        aws_s3_bucket.raw.arn,
        "${aws_s3_bucket.raw.arn}/*"
      ]
    }]
  })
}

# Glue Crawler
resource "aws_glue_crawler" "vendas" {
  database_name = aws_glue_catalog_database.analytics.name
  name          = "${local.prefix}-vendas-crawler"
  role          = aws_iam_role.glue_role.arn

  s3_target {
    path = "s3://${aws_s3_bucket.raw.bucket}/vendas/"
  }

  schema_change_policy {
    update_behavior = "UPDATE_IN_DATABASE"
    delete_behavior = "DEPRECATE_IN_DATABASE"
  }
}

# Athena Workgroup
resource "aws_athena_workgroup" "lab" {
  name          = "${local.prefix}-workgroup"
  force_destroy = true

  configuration {
    result_configuration {
      output_location = "s3://${aws_s3_bucket.athena_results.bucket}/results/"
    }
    bytes_scanned_cutoff_per_query = 1073741824  # 1 GB limit for safety
  }
}

output "raw_bucket" {
  value = aws_s3_bucket.raw.bucket
}

output "glue_database" {
  value = aws_glue_catalog_database.analytics.name
}
```

---

## Part 2 — Sample Data (CSV)

```bash
# Create sample sales data
cat > vendas_2024_01.csv << 'EOF'
pedido_id,data,produto,categoria,quantidade,preco_unitario,regiao
1001,2024-01-15,Notebook,Electronics,1,2500.00,Southeast
1002,2024-01-15,Mouse,Peripherals,3,89.90,South
1003,2024-01-16,Monitor,Electronics,2,1200.00,North
1004,2024-01-16,Keyboard,Peripherals,5,149.90,Southeast
1005,2024-01-17,Notebook,Electronics,2,2500.00,Center-West
1006,2024-01-17,Headset,Peripherals,4,299.90,Northeast
1007,2024-01-18,Monitor,Electronics,1,1200.00,South
1008,2024-01-18,SSD 1TB,Storage,6,399.90,Southeast
1009,2024-01-19,Mouse,Peripherals,2,89.90,North
1010,2024-01-20,Notebook,Electronics,3,2500.00,Southeast
EOF

cat > vendas_2024_02.csv << 'EOF'
pedido_id,data,produto,categoria,quantidade,preco_unitario,regiao
2001,2024-02-01,Monitor,Electronics,4,1200.00,Southeast
2002,2024-02-05,Notebook,Electronics,2,2699.00,South
2003,2024-02-10,Headset,Peripherals,8,299.90,Northeast
2004,2024-02-14,SSD 1TB,Storage,10,399.90,Southeast
2005,2024-02-20,Keyboard,Peripherals,6,149.90,Center-West
EOF
```

---

## Part 3 — Deploy and Data Load

```bash
# 1. Deploy infrastructure
terraform init
terraform apply -auto-approve

# 2. Capture the bucket name
BUCKET=$(terraform output -raw raw_bucket)
echo "Bucket: $BUCKET"

# 3. Upload the CSVs partitioned by date
aws s3 cp vendas_2024_01.csv "s3://${BUCKET}/vendas/year=2024/month=01/vendas_jan.csv"
aws s3 cp vendas_2024_02.csv "s3://${BUCKET}/vendas/year=2024/month=02/vendas_fev.csv"

# 4. Run the Glue Crawler (catalogs the schema automatically)
aws glue start-crawler --name lab-analytics-vendas-crawler

# 5. Wait for the crawler to finish
watch -n 10 'aws glue get-crawler --name lab-analytics-vendas-crawler \
  --query "Crawler.State" --output text'
# Wait until "READY" appears
```

---

## Part 4 — Athena Queries via CLI

```bash
DB="lab_analytics_db"
WORKGROUP="lab-analytics-workgroup"
BUCKET=$(terraform output -raw raw_bucket)
RESULT_BUCKET="lab-analytics-athena-results-$(aws sts get-caller-identity --query Account --output text)"

# Helper function to run queries
run_query() {
  local sql="$1"
  local query_id=$(aws athena start-query-execution \
    --query-string "$sql" \
    --query-execution-context "Database=${DB}" \
    --work-group "${WORKGROUP}" \
    --query 'QueryExecutionId' --output text)
  
  echo "Query ID: $query_id"
  
  # Wait for completion
  while true; do
    status=$(aws athena get-query-execution \
      --query-execution-id "$query_id" \
      --query 'QueryExecution.Status.State' --output text)
    [ "$status" = "SUCCEEDED" ] && break
    [ "$status" = "FAILED" ] && echo "FAILED!" && break
    sleep 2
  done
  
  # Results
  aws athena get-query-results \
    --query-execution-id "$query_id" \
    --query 'ResultSet.Rows[*].Data[*].VarCharValue' \
    --output table
}

# QUERY 1 — Check the table created by the Crawler
run_query "SHOW TABLES IN ${DB}"

# QUERY 2 — Total sales by category
run_query "
SELECT categoria,
       COUNT(*) AS num_pedidos,
       SUM(quantidade) AS total_itens,
       ROUND(SUM(quantidade * preco_unitario), 2) AS receita_total
FROM vendas
GROUP BY categoria
ORDER BY receita_total DESC"

# QUERY 3 — Top 3 regions by revenue
run_query "
SELECT regiao,
       ROUND(SUM(quantidade * preco_unitario), 2) AS receita
FROM vendas
GROUP BY regiao
ORDER BY receita DESC
LIMIT 3"

# QUERY 4 — Using partitions (more efficient — less data scanned)
run_query "
SELECT data, produto, quantidade
FROM vendas
WHERE year = '2024' AND month = '01'
ORDER BY data"

# QUERY 5 — Create a partitioned table in Parquet format (better performance)
run_query "
CREATE TABLE IF NOT EXISTS vendas_parquet
WITH (
  format = 'PARQUET',
  parquet_compression = 'SNAPPY',
  external_location = 's3://${BUCKET}/vendas_parquet/',
  partitioned_by = ARRAY['year', 'month']
) AS
SELECT pedido_id, data, produto, categoria, quantidade, preco_unitario, regiao,
       year, month
FROM vendas"
```

---

## Part 5 — Compare CSV vs Parquet Cost

```bash
# View bytes scanned in the latest queries
aws athena list-query-executions \
  --work-group "${WORKGROUP}" \
  --query 'QueryExecutionIds[0:5]' \
  --output text | tr '\t' '\n' | while read qid; do
    aws athena get-query-execution \
      --query-execution-id "$qid" \
      --query 'QueryExecution.Statistics.DataScannedInBytes'
done

# Cost per query: DataScannedInBytes / 1TB * $5.00
# CSV (raw): full file scan
# Parquet: scans only the required columns (column pruning)
# Typical result: Parquet 10-20x smaller scan = 10-20x cheaper
```

---

## Cleanup

```bash
# Remove S3 files first (force_destroy is enabled)
aws s3 rm "s3://${BUCKET}" --recursive

# Destroy infrastructure
terraform destroy -auto-approve
```

---

## What You Learned

- The Glue Crawler detects the schema automatically and creates a table in the Data Catalog
- Athena charges per bytes scanned — partitioning and using Parquet drastically reduces cost
- Partitioning (year/month) lets Athena filter only the relevant files
- Athena is serverless — zero servers, you pay only for the queries executed
- `bytes_scanned_cutoff_per_query` in the Workgroup protects against accidentally expensive queries
