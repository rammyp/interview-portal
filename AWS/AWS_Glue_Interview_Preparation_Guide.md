# AWS Glue Interview Preparation Guide

---

## Introduction

This guide covers everything you need to prepare for AWS Glue interviews, from fundamental concepts to advanced scenarios. Use this as a structured study plan over 1-2 weeks.

---

## Study Plan Overview

| Focus Area |
|------------|
| Core Concepts & Architecture |
| ETL Development & DynamicFrames |
| Performance Optimization |
| Integration & Security |
| Hands-on Practice & Scenarios |

---

## Section 1: Core Concepts & Architecture

### What is AWS Glue?

AWS Glue is a fully managed, serverless ETL (Extract, Transform, Load) service that simplifies data preparation for analytics. Key characteristics:

- **Serverless** — No infrastructure to provision or manage
- **Pay-per-use** — Charged based on DPU-hours consumed
- **Apache Spark-based** — Uses Spark for distributed processing
- **Integrated** — Works seamlessly with AWS data services

### Main Components

#### 1. Data Catalog
- Central metadata repository
- Stores table definitions, schemas, and partition information
- Compatible with Apache Hive Metastore
- Shared across Athena, Redshift Spectrum, and EMR

#### 2. Crawlers
- Automatically discover data schemas
- Support multiple data stores (S3, JDBC, DynamoDB)
- Detect schema changes and new partitions
- Populate the Data Catalog

#### 3. ETL Jobs
- Transform and move data between sources
- Written in Python (PySpark) or Scala
- Run on managed Spark infrastructure

#### 4. Triggers
- Schedule-based (cron expressions)
- Event-based (job completion, crawler completion)
- On-demand

#### 5. Workflows
- Orchestrate multiple crawlers and jobs
- Define dependencies and execution order
- Visual monitoring in console

### Study Checklist
- [ ] Explain what AWS Glue is and its use cases
- [ ] List and describe all main components
- [ ] Understand how Data Catalog works
- [ ] Know the difference between crawlers and jobs
- [ ] Explain triggers and workflows

---

## Section 2: ETL Development

### Supported Languages

| Language | Use Case |
|----------|----------|
| Python (PySpark) | Most common, general ETL |
| Scala (Spark) | Performance-critical jobs |
| Python Shell | Lightweight tasks, no Spark overhead |

### DynamicFrame vs DataFrame

DynamicFrame is Glue's extension of Spark DataFrame:

| Feature | DataFrame | DynamicFrame |
|---------|-----------|--------------|
| Schema | Fixed upfront | Computed on-the-fly |
| Schema inconsistencies | Fails | Handles gracefully |
| Self-describing | No | Yes (schema per record) |
| Transformations | Spark SQL | Glue transforms + Spark |

#### Converting Between Them
```python
# DynamicFrame to DataFrame
df = dynamic_frame.toDF()

# DataFrame to DynamicFrame
dynamic_frame = DynamicFrame.fromDF(df, glue_context, "name")
```

### Essential DynamicFrame Transformations

#### ApplyMapping
Rename and cast columns:
```python
mapped = ApplyMapping.apply(
    frame=datasource,
    mappings=[
        ("old_name", "string", "new_name", "string"),
        ("price", "string", "price", "double")
    ]
)
```

#### ResolveChoice
Handle ambiguous types:
```python
resolved = datasource.resolveChoice(
    specs=[("price", "cast:double")]
)
# Options: cast, make_struct, make_cols, project
```

#### Filter
Filter records:
```python
filtered = Filter.apply(
    frame=datasource,
    f=lambda x: x["status"] == "active"
)
```

#### DropFields / SelectFields
```python
# Remove specific fields
dropped = DropFields.apply(frame=datasource, paths=["temp_field"])

# Keep only specific fields
selected = SelectFields.apply(frame=datasource, paths=["id", "name"])
```

#### Relationalize
Flatten nested structures:
```python
relationalized = datasource.relationalize(
    "root_table",
    "s3://bucket/temp/"
)
```

### Basic Job Structure
```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glue_context = GlueContext(sc)
spark = glue_context.spark_session
job = Job(glue_context)
job.init(args['JOB_NAME'], args)

# Read from catalog
datasource = glue_context.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="mytable"
)

# Transform
transformed = ApplyMapping.apply(
    frame=datasource,
    mappings=[("id", "long", "id", "long")]
)

# Write
glue_context.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={"path": "s3://bucket/output/"},
    format="parquet"
)

job.commit()
```

### Study Checklist
- [ ] Understand DynamicFrame vs DataFrame differences
- [ ] Know all major transformations and when to use them
- [ ] Write a basic Glue job from scratch
- [ ] Convert between DynamicFrame and DataFrame
- [ ] Handle schema inconsistencies with ResolveChoice

---

## Section 3: Performance Optimization

### Worker Types

| Type | vCPU | Memory | Disk | Best For |
|------|------|--------|------|----------|
| Standard | 4 | 16 GB | 50 GB | General workloads |
| G.1X | 4 | 16 GB | 64 GB | Memory-intensive |
| G.2X | 8 | 32 GB | 128 GB | ML transforms, large shuffles |
| G.4X | 16 | 64 GB | 256 GB | Very large datasets |
| G.8X | 32 | 128 GB | 512 GB | Extremely large workloads |

### Job Bookmarks

Track processed data to enable incremental loads:

```python
job.init(args['JOB_NAME'], args)

# Your ETL logic here

job.commit()  # Required to update bookmark
```

**Enable in job configuration:**
- Job bookmark: Enabled
- Tracks: S3 paths, JDBC timestamps, partition keys

### Pushdown Predicates

Filter data at the source to reduce data scanned:

```python
# S3/Catalog source
datasource = glue_context.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="mytable",
    push_down_predicate="year='2024' and month='01'"
)

# JDBC source
datasource = glue_context.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="jdbc_table",
    additional_options={
        "pushDownPredicate": "created_date > '2024-01-01'"
    }
)
```

### Partitioning Strategies

**Reading partitioned data:**
```python
# Catalog automatically recognizes partitions
datasource = glue_context.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="partitioned_table",
    push_down_predicate="year='2024'"
)
```

**Writing partitioned data:**
```python
glue_context.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={
        "path": "s3://bucket/output/",
        "partitionKeys": ["year", "month"]
    },
    format="parquet"
)
```

### File Format Best Practices

| Format | Compression | Use Case |
|--------|-------------|----------|
| Parquet | Snappy (default) | Analytics, columnar queries |
| ORC | Zlib | Hive ecosystem |
| JSON | Gzip | Semi-structured, debugging |
| CSV | Gzip | Legacy systems, simple data |

### Optimization Checklist

- [ ] Choose appropriate worker type for workload
- [ ] Enable job bookmarks for incremental loads
- [ ] Use pushdown predicates to filter at source
- [ ] Partition output data appropriately
- [ ] Use columnar formats (Parquet/ORC)
- [ ] Right-size number of workers
- [ ] Coalesce small files before writing

---

## Section 4: Integration & Security

### AWS Glue and Athena Integration

Glue and Athena integrate primarily through the **shared Data Catalog**:

```
┌─────────────┐      ┌──────────────────┐      ┌─────────────┐
│   S3 Data   │ ──── │  Glue Crawler    │ ──── │ Data Catalog│
└─────────────┘      └──────────────────┘      └──────┬──────┘
                                                      │
                            ┌─────────────────────────┼─────────────────────────┐
                            │                         │                         │
                            ▼                         ▼                         ▼
                     ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
                     │   Athena    │          │  Glue ETL   │          │   Redshift  │
                     │  (Query)    │          │  (Transform)│          │  Spectrum   │
                     └─────────────┘          └─────────────┘          └─────────────┘
```

#### Typical Workflow

**1. Raw Data Lands in S3**
```
s3://my-bucket/raw-data/
├── 2024/01/data.json
├── 2024/02/data.json
└── 2024/03/data.json
```

**2. Glue Crawler Discovers Schema**
- Scans S3 location
- Infers schema and partitions
- Creates table in Data Catalog

**3. Athena Queries the Data**
```sql
-- Athena uses the table created by Glue
SELECT * FROM my_database.raw_data
WHERE year = '2024' AND month = '01'
```

**4. Glue ETL Transforms Data**
```python
# Read from catalog (same table Athena uses)
source = glue_context.create_dynamic_frame.from_catalog(
    database="my_database",
    table_name="raw_data"
)

# Transform and write to processed location
glue_context.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={
        "path": "s3://my-bucket/processed/",
        "partitionKeys": ["year", "month"]
    },
    format="parquet"
)
```

**5. Crawler Updates Catalog with Processed Data**

**6. Athena Queries Optimized Data**
```sql
-- Query the transformed Parquet data
SELECT customer_id, SUM(amount)
FROM my_database.processed_data
GROUP BY customer_id
```

#### Key Integration Points

| Component | Role |
|-----------|------|
| **Glue Crawler** | Discovers schema, populates catalog |
| **Glue Data Catalog** | Shared metadata store |
| **Glue ETL** | Transforms raw → optimized formats |
| **Athena** | Ad-hoc SQL queries on cataloged data |

#### Common Use Cases

- **Data Lake Analytics** — Glue crawls and catalogs raw data; Athena provides instant SQL access
- **ETL + Query Pipeline** — Glue transforms JSON → Parquet; Athena queries optimized Parquet
- **Schema Management** — Glue maintains schema versions; Athena automatically uses latest schema
- **Partition Management** — Glue discovers partitions; Athena uses them for query optimization

#### Benefits of Using Together

- **No data movement** — Both read from S3
- **Single metadata store** — No schema duplication
- **Cost-effective** — Serverless, pay-per-use
- **Separation of concerns** — Glue for ETL, Athena for queries

---

### Supported Data Sources

**AWS Services:**
- Amazon S3
- Amazon RDS (all engines)
- Amazon Redshift
- Amazon DynamoDB
- Amazon DocumentDB
- Amazon Kinesis (streaming)
- Amazon MSK/Kafka (streaming)

**JDBC Sources:**
- MySQL
- PostgreSQL
- Oracle
- SQL Server
- Custom JDBC drivers

### Integration Patterns

#### With Amazon Athena
- Shared Data Catalog
- Crawlers populate tables for Athena queries
- No data movement required

#### With Amazon Redshift
```python
# Write to Redshift
glue_context.write_dynamic_frame.from_jdbc_conf(
    frame=transformed,
    catalog_connection="redshift-connection",
    connection_options={
        "dbtable": "schema.table",
        "database": "mydb"
    },
    redshift_tmp_dir="s3://bucket/temp/"
)
```

#### With AWS Lambda
- Trigger Glue jobs from Lambda
- Process event-driven workflows
- Handle S3 event notifications

#### With Step Functions
- Complex workflow orchestration
- Error handling and retries
- Parallel execution

### VPC Configuration

Connect to resources in private VPCs:

1. Create a Glue Connection with VPC settings
2. Specify subnet and security groups
3. Glue creates ENIs for private access
4. Ensure security groups allow required traffic

### Security Best Practices

#### IAM Roles
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "glue:*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Encryption
- **At rest:** SSE-S3, SSE-KMS for S3 data
- **In transit:** SSL/TLS for all connections
- **Job artifacts:** KMS encryption available

#### Data Catalog Security
- Resource-based policies
- Fine-grained access control
- Cross-account sharing

### Security Checklist
- [ ] Configure appropriate IAM roles
- [ ] Set up VPC connections for private resources
- [ ] Enable encryption at rest and in transit
- [ ] Implement Data Catalog access policies
- [ ] Use AWS Secrets Manager for credentials

---

## Section 5: Advanced Topics

### Glue Streaming

Process streaming data from Kinesis or Kafka:

```python
from awsglue.context import GlueContext
from pyspark.context import SparkContext

sc = SparkContext()
glue_context = GlueContext(sc)

# Read from Kinesis
streaming_df = glue_context.create_data_frame.from_catalog(
    database="mydb",
    table_name="kinesis_stream",
    additional_options={"startingPosition": "TRIM_HORIZON"}
)

# Process with foreachBatch
def process_batch(batch_df, batch_id):
    # Transform and write each micro-batch
    pass

streaming_df.writeStream \
    .foreachBatch(process_batch) \
    .start()
```

### Glue DataBrew

Visual data preparation tool:
- No-code interface
- 250+ built-in transformations
- Data profiling and quality checks
- Recipe-based transformations

### Machine Learning Transforms

**FindMatches:** Identify duplicate records
```python
# Train ML model to find matches
findmatches = FindMatches(
    frame=datasource,
    transformId="transform-id"
)
```

### Error Handling

```python
try:
    # ETL logic
    datasource = glue_context.create_dynamic_frame.from_catalog(...)
except Exception as e:
    # Log error
    print(f"Error: {str(e)}")
    # Optionally write to dead-letter location
    raise e
finally:
    job.commit()
```

### Monitoring

**CloudWatch Metrics:**
- Job duration
- DPU hours consumed
- Bytes read/written
- Record counts

**CloudWatch Logs:**
- Driver logs
- Executor logs
- Spark UI logs

---

## Section 6: Scenario-Based Questions

### Scenario 1: Incremental Data Loading

**Problem:** Process only new data arriving daily in S3.

**Solution:**
```python
# Enable job bookmarks in job configuration
job.init(args['JOB_NAME'], args)

datasource = glue_context.create_dynamic_frame.from_catalog(
    database="mydb",
    table_name="daily_data",
    transformation_ctx="datasource"  # Required for bookmarks
)

# Process and write
glue_context.write_dynamic_frame.from_options(
    frame=transformed,
    connection_type="s3",
    connection_options={"path": "s3://output/"},
    format="parquet",
    transformation_ctx="output"  # Required for bookmarks
)

job.commit()  # Updates bookmark
```

### Scenario 2: Handling Schema Evolution

**Problem:** Source data schema changes over time.

**Solution:**
```python
# DynamicFrames handle this naturally
datasource = glue_context.create_dynamic_frame.from_catalog(...)

# Resolve any choice types
resolved = datasource.resolveChoice(
    choice="make_struct"  # Or cast to specific type
)

# Map to consistent output schema
mapped = ApplyMapping.apply(
    frame=resolved,
    mappings=[
        ("id", "long", "id", "long"),
        ("name", "string", "name", "string"),
        ("value", "double", "value", "double")
    ]
)
```

### Scenario 3: Large File Optimization

**Problem:** Many small files causing slow performance.

**Solution:**
```python
# Convert DynamicFrame to DataFrame for repartitioning
df = datasource.toDF()

# Coalesce to optimal number of files
df_optimized = df.coalesce(10)  # Adjust based on data size

# Convert back and write
output = DynamicFrame.fromDF(df_optimized, glue_context, "output")

glue_context.write_dynamic_frame.from_options(
    frame=output,
    connection_type="s3",
    connection_options={"path": "s3://output/"},
    format="parquet"
)
```

### Scenario 4: Cross-Account Data Access

**Problem:** Read data from another AWS account's S3 bucket.

**Solution:**
1. Set up cross-account IAM role
2. Configure bucket policy in source account
3. Use assumed role in Glue job

```python
import boto3

# Assume cross-account role
sts = boto3.client('sts')
assumed = sts.assume_role(
    RoleArn='arn:aws:iam::TARGET_ACCOUNT:role/CrossAccountRole',
    RoleSessionName='GlueSession'
)

# Use credentials for S3 access
```

---

## Section 7: Comparison Questions

### Using EMR to Refresh Tables on New Data

EMR can refresh tables when new data arrives using several approaches:

#### 1. Partition Refresh (Hive/Spark SQL)
```sql
-- Discover new partitions automatically
MSCK REPAIR TABLE my_database.my_table;

-- Or add specific partition
ALTER TABLE my_database.my_table 
ADD PARTITION (year='2024', month='12') 
LOCATION 's3://bucket/data/year=2024/month=12/';
```

#### 2. Incremental Data Load (Spark)
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("IncrementalLoad") \
    .enableHiveSupport() \
    .getOrCreate()

# Read new data
new_data = spark.read.parquet("s3://bucket/incoming/")

# Append to existing table
new_data.write \
    .mode("append") \
    .partitionBy("year", "month") \
    .saveAsTable("my_database.my_table")
```

#### 3. Upsert with Delta Lake / Iceberg
```python
# Delta Lake MERGE (upsert)
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "s3://bucket/delta-table/")

delta_table.alias("target").merge(
    new_data.alias("source"),
    "target.id = source.id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```

```python
# Apache Iceberg MERGE
spark.sql("""
    MERGE INTO catalog.db.my_table t
    USING new_data s
    ON t.id = s.id
    WHEN MATCHED THEN UPDATE SET *
    WHEN NOT MATCHED THEN INSERT *
""")
```

#### 4. Triggering EMR on New Data

```
┌──────────┐     ┌──────────┐     ┌───────────────┐     ┌──────────┐
│ S3 Event │ ──▶ │  Lambda  │ ──▶ │Step Functions │ ──▶ │   EMR    │
└──────────┘     └──────────┘     └───────────────┘     └──────────┘
```

**Lambda + EMR Steps:**
```python
import boto3

def lambda_handler(event, context):
    emr = boto3.client('emr')
    
    emr.add_job_flow_steps(
        JobFlowId='j-XXXXXXXXXXXXX',
        Steps=[{
            'Name': 'Refresh Table',
            'ActionOnFailure': 'CONTINUE',
            'HadoopJarStep': {
                'Jar': 'command-runner.jar',
                'Args': ['spark-submit', 's3://bucket/scripts/refresh.py']
            }
        }]
    )
```

#### EMR vs Glue for Table Refresh

| Aspect | EMR | Glue |
|--------|-----|------|
| Startup time | 5-15 min (cluster) | Seconds (serverless) |
| Best for | Long-running, complex jobs | Quick ETL tasks |
| Cost model | EC2 instance hours | DPU hours |
| Customization | Full control | Limited |
| Delta/Iceberg support | Native | Limited |

#### When to Use Which for Table Refresh

| Scenario | Best Choice |
|----------|-------------|
| Frequent small updates | Glue (faster startup) |
| Large batch processing | EMR |
| ACID transactions needed | EMR with Delta/Iceberg |
| Already have EMR cluster | EMR Steps |
| Serverless preference | Glue |

---

### Glue vs EMR

| Aspect | AWS Glue | Amazon EMR |
|--------|----------|------------|
| Management | Fully managed | User-managed clusters |
| Pricing | Per DPU-hour | Per EC2 instance-hour |
| Setup time | Minutes | 10-15 minutes |
| Customization | Limited | Full control |
| Use cases | ETL jobs | Complex analytics, ML |
| Spark version | Fixed per Glue version | User choice |

**Choose Glue when:**
- Quick ETL jobs
- Serverless preference
- Limited Spark customization needed

**Choose EMR when:**
- Complex processing requirements
- Need specific Spark/Hadoop versions
- Long-running clusters
- ML workloads

### Glue vs Data Pipeline

| Aspect | AWS Glue | AWS Data Pipeline |
|--------|----------|-------------------|
| Focus | ETL/Spark | Workflow orchestration |
| Compute | Managed Spark | EC2/EMR |
| Scheduling | Basic | Advanced |
| Status | Active development | Maintenance mode |

### Glue ETL vs Glue DataBrew

| Aspect | Glue ETL | Glue DataBrew |
|--------|----------|---------------|
| Interface | Code-based | Visual/No-code |
| Flexibility | High | Moderate |
| Users | Data engineers | Data analysts |
| Learning curve | Steeper | Gentle |

---

## Section 8: Quick Reference

### Common CLI Commands

```bash
# Start a job run
aws glue start-job-run --job-name my-job

# Get job run status
aws glue get-job-run --job-name my-job --run-id jr_xxx

# Start a crawler
aws glue start-crawler --name my-crawler

# Get crawler status
aws glue get-crawler --name my-crawler
```

### Key Limits

| Resource | Default Limit |
|----------|---------------|
| Max DPUs per job | 100 |
| Max concurrent job runs | 50 |
| Max tables per database | 200,000 |
| Max partitions per table | 10,000,000 |
| Crawler timeout | 48 hours |

### Pricing Components

- **ETL jobs:** $0.44 per DPU-hour
- **Data Catalog:** Free for first million objects
- **Crawlers:** $0.44 per DPU-hour
- **DataBrew:** $1.00 per session

---

## Final Preparation Tips

1. **Hands-on practice:** Create sample jobs in AWS Free Tier
2. **Read documentation:** Focus on best practices guides
3. **Understand trade-offs:** Know when to use Glue vs alternatives
4. **Review error scenarios:** Common errors and troubleshooting
5. **Stay current:** Check for new features and updates

### Resources

- AWS Glue Documentation: https://docs.aws.amazon.com/glue/
- AWS Glue Best Practices: https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-best-practices.html
- AWS Big Data Blog: https://aws.amazon.com/blogs/big-data/
- AWS Skill Builder: Free Glue courses

---

*Good luck with your interview!*
