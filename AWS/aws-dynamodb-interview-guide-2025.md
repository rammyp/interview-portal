# AWS DynamoDB Interview Preparation - Complete Guide 2025

## Table of Contents
1. [Introduction](#1-introduction)
2. [Core Concepts](#2-core-concepts)
3. [Primary Keys](#3-primary-keys)
4. [Secondary Indexes](#4-secondary-indexes)
5. [Capacity Modes](#5-capacity-modes)
6. [Read Consistency](#6-read-consistency)
7. [DynamoDB Operations](#7-dynamodb-operations)
8. [Advanced Features](#8-advanced-features)
9. [Data Modeling](#9-data-modeling)
10. [Security](#10-security)
11. [Performance Optimization](#11-performance-optimization)
12. [Common Interview Questions](#12-common-interview-questions)
13. [Code Examples](#13-code-examples)
14. [Quick Reference](#14-quick-reference)

---

# 1. Introduction

## What is DynamoDB?

**Amazon DynamoDB** is a fully managed, serverless, NoSQL database service that provides fast and predictable performance with seamless scalability.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Type** | NoSQL (Key-Value & Document) |
| **Managed** | Fully managed by AWS (no servers to manage) |
| **Scalability** | Automatic, virtually unlimited |
| **Performance** | Single-digit millisecond latency |
| **Availability** | 99.99% SLA (99.999% with Global Tables) |
| **Storage** | Unlimited |
| **Data Model** | Schemaless (flexible attributes) |

### When to Use DynamoDB

```
✅ USE DYNAMODB FOR:
─────────────────────
• High-scale applications (millions of requests/sec)
• Key-value and document data
• Predictable, low-latency performance
• Serverless architectures
• Gaming leaderboards
• Session management
• IoT data storage
• Shopping carts
• User profiles

❌ AVOID DYNAMODB FOR:
──────────────────────
• Complex joins and relationships
• Ad-hoc queries on any column
• OLAP/Analytics workloads
• Small datasets with complex queries
• Applications requiring strong schema
• Multi-item ACID transactions (use sparingly)
```

---

# 2. Core Concepts

## Terminology

| Term | Description | Analogy (RDBMS) |
|------|-------------|-----------------|
| **Table** | Collection of items | Table |
| **Item** | Single data record | Row |
| **Attribute** | Data element within item | Column |
| **Primary Key** | Unique identifier for item | Primary Key |
| **Partition Key** | Hash key for distribution | N/A |
| **Sort Key** | Range key for ordering | N/A |

## Data Types

### Scalar Types
```
S  - String
N  - Number
B  - Binary
BOOL - Boolean
NULL - Null
```

### Document Types
```
M  - Map (nested attributes)
L  - List (ordered collection)
```

### Set Types
```
SS - String Set
NS - Number Set
BS - Binary Set
```

### Example Item

```json
{
  "UserID": {"S": "user123"},
  "Name": {"S": "John Doe"},
  "Age": {"N": "30"},
  "IsActive": {"BOOL": true},
  "Address": {
    "M": {
      "Street": {"S": "123 Main St"},
      "City": {"S": "New York"},
      "Zip": {"S": "10001"}
    }
  },
  "Tags": {"SS": ["developer", "aws", "dynamodb"]},
  "Orders": {
    "L": [
      {"S": "order1"},
      {"S": "order2"}
    ]
  }
}
```

---

# 3. Primary Keys

## Types of Primary Keys

### 1. Simple Primary Key (Partition Key Only)

```
┌─────────────────────────────────────────────────────┐
│                  PARTITION KEY ONLY                  │
├─────────────────────────────────────────────────────┤
│  • Single attribute                                  │
│  • Must be unique for each item                     │
│  • Determines physical partition                     │
│  • No range queries possible                        │
└─────────────────────────────────────────────────────┘

Example: Users Table
┌───────────────┬─────────────────┬─────────────────┐
│  UserID (PK)  │      Name       │      Email      │
├───────────────┼─────────────────┼─────────────────┤
│  user001      │  John Doe       │  john@mail.com  │
│  user002      │  Jane Smith     │  jane@mail.com  │
│  user003      │  Bob Wilson     │  bob@mail.com   │
└───────────────┴─────────────────┴─────────────────┘
```

### 2. Composite Primary Key (Partition Key + Sort Key)

```
┌─────────────────────────────────────────────────────┐
│            PARTITION KEY + SORT KEY                  │
├─────────────────────────────────────────────────────┤
│  • Two attributes combined                          │
│  • Partition key can repeat                         │
│  • PK + SK combination must be unique               │
│  • Enables range queries on sort key                │
│  • Items with same PK stored together               │
└─────────────────────────────────────────────────────┘

Example: Orders Table
┌───────────────┬───────────────┬─────────┬─────────┐
│  UserID (PK)  │  OrderID (SK) │ Amount  │ Status  │
├───────────────┼───────────────┼─────────┼─────────┤
│  user001      │  order001     │  99.99  │ shipped │
│  user001      │  order002     │  149.99 │ pending │
│  user001      │  order003     │  29.99  │ delivered│
│  user002      │  order001     │  199.99 │ shipped │
└───────────────┴───────────────┴─────────┴─────────┘

Query: Get all orders for user001 → Returns 3 items
Query: Get user001's order002 → Returns 1 item
```

## Partition Key Design Best Practices

### Good Partition Keys ✅

```
• High cardinality (many unique values)
• Uniform access patterns
• Natural identifier

Examples:
─────────
UserID      → Millions of unique users
OrderID     → Each order is unique
SessionID   → Each session is unique
DeviceID    → Many IoT devices
```

### Bad Partition Keys ❌

```
• Low cardinality (few unique values)
• Uneven access patterns (hot partitions)
• Time-based without modification

Examples:
─────────
Status       → Only "Active"/"Inactive" (2 values!)
Country      → Few countries, uneven distribution
Date         → Many items on same date
Boolean      → Only true/false
```

## How Partitions Work

```
                    Hash Function
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
┌──────────┐      ┌──────────┐      ┌──────────┐
│Partition │      │Partition │      │Partition │
│    1     │      │    2     │      │    3     │
├──────────┤      ├──────────┤      ├──────────┤
│ UserA    │      │ UserD    │      │ UserG    │
│ UserB    │      │ UserE    │      │ UserH    │
│ UserC    │      │ UserF    │      │ UserI    │
└──────────┘      └──────────┘      └──────────┘

• Each partition can store ~10GB
• Each partition: 3000 RCU, 1000 WCU
• DynamoDB automatically manages partitions
```

---

# 4. Secondary Indexes

## Global Secondary Index (GSI)

```
┌─────────────────────────────────────────────────────────────┐
│                  GLOBAL SECONDARY INDEX                      │
├─────────────────────────────────────────────────────────────┤
│  • Different partition key AND/OR sort key from base table  │
│  • Spans ALL partitions globally                            │
│  • Has its OWN provisioned throughput                       │
│  • Eventually consistent reads ONLY                          │
│  • Can be created/deleted anytime                           │
│  • Maximum 20 GSIs per table                                │
│  • Can project all, keys only, or specific attributes       │
└─────────────────────────────────────────────────────────────┘

Example:
─────────
Base Table:
  PK = UserID
  SK = OrderID

GSI (Orders by Status):
  PK = Status
  SK = OrderDate

Now you can efficiently query: "Get all PENDING orders from last week"
```

### GSI Projection Types

```
KEYS_ONLY
─────────
• Only key attributes
• Smallest size
• Must fetch base table for other attributes

INCLUDE
───────
• Key attributes + specified attributes
• Balance between size and functionality

ALL
───
• All attributes from base table
• Largest size
• No need to fetch base table
```

## Local Secondary Index (LSI)

```
┌─────────────────────────────────────────────────────────────┐
│                   LOCAL SECONDARY INDEX                      │
├─────────────────────────────────────────────────────────────┤
│  • SAME partition key, DIFFERENT sort key                   │
│  • Scoped to single partition (local)                       │
│  • SHARES throughput with base table                        │
│  • Supports strongly consistent reads                        │
│  • MUST be created at table creation time                   │
│  • Maximum 5 LSIs per table                                 │
│  • 10GB limit per partition key value                       │
└─────────────────────────────────────────────────────────────┘

Example:
─────────
Base Table:
  PK = UserID
  SK = OrderID

LSI (Orders by Date):
  PK = UserID (same)
  SK = OrderDate (different)

Now you can query: "Get user001's orders sorted by date"
```

## GSI vs LSI Comparison

| Feature | GSI | LSI |
|---------|-----|-----|
| **Partition Key** | Can be different | Must be same |
| **Sort Key** | Can be different | Must be different |
| **Scope** | Entire table | Single partition |
| **Throughput** | Separate capacity | Shared with table |
| **Read Consistency** | Eventually consistent only | Strong or eventual |
| **Creation Time** | Anytime | Table creation only |
| **Limit** | 20 per table | 5 per table |
| **Size Limit** | No limit | 10GB per partition |
| **Backfill** | Yes (async) | N/A |

## When to Use Which Index

```
USE GSI WHEN:
─────────────
✓ Need different partition key
✓ Query patterns span multiple partitions
✓ Can tolerate eventual consistency
✓ Need to add index to existing table

USE LSI WHEN:
─────────────
✓ Need different sort key, same partition key
✓ Need strongly consistent reads on index
✓ Know all access patterns at table creation
✓ Query within single partition
```

---

# 5. Capacity Modes

## On-Demand Mode

```
┌─────────────────────────────────────────────────────────────┐
│                    ON-DEMAND MODE                            │
├─────────────────────────────────────────────────────────────┤
│  Pricing: Pay per request                                    │
│                                                              │
│  PROS:                                                       │
│  ✓ No capacity planning needed                              │
│  ✓ Instant scaling (thousands of requests/sec)              │
│  ✓ Perfect for unpredictable workloads                      │
│  ✓ No throttling (within account limits)                    │
│  ✓ Good for new applications                                │
│                                                              │
│  CONS:                                                       │
│  ✗ More expensive for steady workloads                      │
│  ✗ No reserved capacity discounts                           │
│  ✗ 2x previous peak as initial capacity                     │
└─────────────────────────────────────────────────────────────┘

Cost: ~$1.25 per million write requests
      ~$0.25 per million read requests
```

## Provisioned Mode

```
┌─────────────────────────────────────────────────────────────┐
│                   PROVISIONED MODE                           │
├─────────────────────────────────────────────────────────────┤
│  Pricing: Per hour based on RCU/WCU provisioned             │
│                                                              │
│  PROS:                                                       │
│  ✓ Cheaper for predictable workloads                        │
│  ✓ Reserved capacity discounts (up to 77%)                  │
│  ✓ Auto Scaling available                                   │
│  ✓ Better cost control                                      │
│                                                              │
│  CONS:                                                       │
│  ✗ Requires capacity planning                               │
│  ✗ Throttling if capacity exceeded                          │
│  ✗ Under-provisioning = errors                              │
│  ✗ Over-provisioning = wasted money                         │
└─────────────────────────────────────────────────────────────┘

RCU = Read Capacity Unit
WCU = Write Capacity Unit
```

## Capacity Unit Calculations

### Read Capacity Units (RCU)

```
┌─────────────────────────────────────────────────────────────┐
│                READ CAPACITY UNITS                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1 RCU = 1 Strongly Consistent Read up to 4 KB/sec          │
│        = 2 Eventually Consistent Reads up to 4 KB/sec       │
│        = 0.5 Transactional Reads up to 4 KB/sec             │
│                                                              │
│  Formula: RCUs = ⌈Item Size / 4 KB⌉ × Reads/sec             │
│           (Divide by 2 for eventually consistent)            │
│           (Multiply by 2 for transactional)                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘

EXAMPLES:
─────────

Example 1: Strongly Consistent Reads
  • 100 reads/second
  • Item size: 6 KB
  • RCUs = ⌈6/4⌉ × 100 = 2 × 100 = 200 RCUs

Example 2: Eventually Consistent Reads
  • 100 reads/second
  • Item size: 6 KB
  • RCUs = ⌈6/4⌉ × 100 / 2 = 2 × 100 / 2 = 100 RCUs

Example 3: Transactional Reads
  • 100 reads/second
  • Item size: 6 KB
  • RCUs = ⌈6/4⌉ × 100 × 2 = 2 × 100 × 2 = 400 RCUs
```

### Write Capacity Units (WCU)

```
┌─────────────────────────────────────────────────────────────┐
│               WRITE CAPACITY UNITS                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1 WCU = 1 Write up to 1 KB/sec                             │
│        = 0.5 Transactional Write up to 1 KB/sec             │
│                                                              │
│  Formula: WCUs = ⌈Item Size / 1 KB⌉ × Writes/sec            │
│           (Multiply by 2 for transactional)                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘

EXAMPLES:
─────────

Example 1: Standard Writes
  • 50 writes/second
  • Item size: 2.5 KB
  • WCUs = ⌈2.5/1⌉ × 50 = 3 × 50 = 150 WCUs

Example 2: Transactional Writes
  • 50 writes/second
  • Item size: 2.5 KB
  • WCUs = ⌈2.5/1⌉ × 50 × 2 = 3 × 50 × 2 = 300 WCUs
```

### Calculation Practice Problems

```
PROBLEM 1:
──────────
Application needs:
• 500 strongly consistent reads/second
• Each item is 12 KB
• 200 writes/second
• Each write is 3 KB

Solution:
• RCUs = ⌈12/4⌉ × 500 = 3 × 500 = 1500 RCUs
• WCUs = ⌈3/1⌉ × 200 = 3 × 200 = 600 WCUs


PROBLEM 2:
──────────
Application needs:
• 1000 eventually consistent reads/second
• Each item is 8 KB
• 100 transactional writes/second
• Each write is 2 KB

Solution:
• RCUs = ⌈8/4⌉ × 1000 / 2 = 2 × 500 = 1000 RCUs
• WCUs = ⌈2/1⌉ × 100 × 2 = 2 × 100 × 2 = 400 WCUs
```

## Auto Scaling

```
┌─────────────────────────────────────────────────────────────┐
│                    AUTO SCALING                              │
├─────────────────────────────────────────────────────────────┤
│  • Works with Provisioned mode                              │
│  • Uses Application Auto Scaling service                    │
│  • Set minimum, maximum capacity                            │
│  • Set target utilization (e.g., 70%)                       │
│  • Scales within 1-2 minutes                                │
│  • Can scale GSI independently                              │
└─────────────────────────────────────────────────────────────┘

Configuration:
• Minimum capacity: 5 RCUs / 5 WCUs
• Maximum capacity: 1000 RCUs / 1000 WCUs
• Target utilization: 70%

Behavior:
• Usage > 70% → Scale up
• Usage < 70% → Scale down (after cooldown)
```

---

# 6. Read Consistency

## Consistency Models

```
┌─────────────────────────────────────────────────────────────┐
│              EVENTUALLY CONSISTENT (Default)                 │
├─────────────────────────────────────────────────────────────┤
│  • May not reflect latest write                             │
│  • Data consistent within ~1 second                         │
│  • Uses 0.5 RCU per 4 KB                                    │
│  • Lower latency                                            │
│  • Best for: Most read scenarios                            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 STRONGLY CONSISTENT                          │
├─────────────────────────────────────────────────────────────┤
│  • Always returns latest data                               │
│  • Uses 1 RCU per 4 KB                                      │
│  • Slightly higher latency                                  │
│  • Not available on GSI                                     │
│  • Best for: Critical data requiring latest value           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   TRANSACTIONAL                              │
├─────────────────────────────────────────────────────────────┤
│  • ACID compliant                                           │
│  • Uses 2 RCU per 4 KB                                      │
│  • Higher latency                                           │
│  • Best for: Multi-item operations needing atomicity        │
└─────────────────────────────────────────────────────────────┘
```

## Visual Comparison

```
                     Write                    Read
                       │                        │
                       ▼                        ▼
               ┌───────────────┐        ┌─────────────┐
               │   Primary     │        │   Request   │
               │   Partition   │        │   arrives   │
               └───────┬───────┘        └──────┬──────┘
                       │                       │
         ┌─────────────┼─────────────┐         │
         ▼             ▼             ▼         │
    ┌─────────┐  ┌─────────┐  ┌─────────┐     │
    │Replica 1│  │Replica 2│  │Replica 3│     │
    │ (sync)  │  │ (async) │  │ (async) │     │
    └─────────┘  └─────────┘  └─────────┘     │
                                               │
                    ┌──────────────────────────┘
                    │
          ┌─────────┴─────────┐
          │                   │
    Eventually           Strongly
    Consistent           Consistent
          │                   │
    (Any replica)       (Primary)
          │                   │
    (Might be stale)    (Always latest)
```

---

# 7. DynamoDB Operations

## Core API Operations

### Item Operations

| Operation | Description | Use Case |
|-----------|-------------|----------|
| `GetItem` | Read single item by PK | Get user by UserID |
| `PutItem` | Create or replace item | Create new user |
| `UpdateItem` | Modify attributes | Update user email |
| `DeleteItem` | Remove item | Delete user |

### Query & Scan

| Operation | Description | Use Case |
|-----------|-------------|----------|
| `Query` | Find items by PK (+ optional SK) | Get user's orders |
| `Scan` | Read entire table | Find all items |

### Batch Operations

| Operation | Description | Limit |
|-----------|-------------|-------|
| `BatchGetItem` | Read multiple items | 100 items, 16 MB |
| `BatchWriteItem` | Write/delete multiple items | 25 items, 16 MB |

### Transaction Operations

| Operation | Description | Limit |
|-----------|-------------|-------|
| `TransactGetItems` | Atomic read of multiple items | 100 items, 4 MB |
| `TransactWriteItems` | Atomic write of multiple items | 100 items, 4 MB |

## Query vs Scan

```
┌─────────────────────────────────────────────────────────────┐
│                         QUERY                                │
├─────────────────────────────────────────────────────────────┤
│  ✅ Efficient - reads only matching partitions              │
│  ✅ Requires partition key                                  │
│  ✅ Can use sort key conditions (=, <, >, BETWEEN, BEGINS)  │
│  ✅ Returns items in sort key order                         │
│  ✅ Supports FilterExpression for additional filtering      │
│  ✅ Can query indexes (GSI/LSI)                             │
│                                                              │
│  Use Case: Get all orders for specific user                 │
│  Cost: Reads only relevant items                            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                          SCAN                                │
├─────────────────────────────────────────────────────────────┤
│  ❌ Inefficient - reads ENTIRE table                        │
│  ❌ Consumes lots of capacity                               │
│  ❌ Slow for large tables                                   │
│  ✅ No partition key required                               │
│  ✅ Can use FilterExpression                                │
│  ✅ Parallel scan available for speed                       │
│                                                              │
│  Use Case: One-time migration, finding all items            │
│  Cost: Reads every item in table                            │
└─────────────────────────────────────────────────────────────┘
```

## Expression Attributes

### Key Condition Expression (Query only)

```python
# Query with sort key condition
KeyConditionExpression='UserID = :uid AND OrderDate BETWEEN :start AND :end'

# Supported operators for sort key:
# =, <, <=, >, >=, BETWEEN, begins_with
```

### Filter Expression

```python
# Filter results AFTER query/scan (still consumes RCUs!)
FilterExpression='Status = :status AND Amount > :amount'

# Does NOT reduce RCUs consumed
# Only filters results returned
```

### Projection Expression

```python
# Return only specific attributes
ProjectionExpression='UserID, #n, Email'

# Reduces data transfer
# Use ExpressionAttributeNames for reserved words
ExpressionAttributeNames={'#n': 'Name'}
```

### Update Expression

```python
# SET - add or update attributes
UpdateExpression='SET #n = :name, Age = Age + :inc'

# REMOVE - delete attributes
UpdateExpression='REMOVE OldAttribute'

# ADD - add to number or set
UpdateExpression='ADD ViewCount :inc, Tags :newTags'

# DELETE - remove from set
UpdateExpression='DELETE Tags :tagsToRemove'
```

### Condition Expression

```python
# Only update if condition is true
ConditionExpression='attribute_exists(UserID) AND Version = :expected'

# Optimistic locking pattern
# Prevents overwriting concurrent changes
```

---

# 8. Advanced Features

## DynamoDB Streams

```
┌─────────────────────────────────────────────────────────────┐
│                   DynamoDB Streams                           │
├─────────────────────────────────────────────────────────────┤
│  • Captures item-level modifications                        │
│  • Time-ordered sequence of changes                         │
│  • 24-hour retention                                        │
│  • Near real-time (typically < 1 second)                    │
│  • Integrates with AWS Lambda                               │
│  • No impact on table performance                           │
└─────────────────────────────────────────────────────────────┘

Events captured:
• INSERT - New item added
• MODIFY - Existing item updated
• REMOVE - Item deleted
```

### Stream View Types

| View Type | Content | Use Case |
|-----------|---------|----------|
| `KEYS_ONLY` | Only key attributes | Lightweight triggers |
| `NEW_IMAGE` | Entire item after change | Replication |
| `OLD_IMAGE` | Entire item before change | Audit trail |
| `NEW_AND_OLD_IMAGES` | Both versions | Change comparison |

### Stream Use Cases

```
┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│    DynamoDB      │────►│    Streams      │────►│     Lambda       │
│     Table        │     │                 │     │                  │
└──────────────────┘     └─────────────────┘     └────────┬─────────┘
                                                          │
                    ┌─────────────────────────────────────┼────────────────────┐
                    │                                     │                    │
                    ▼                                     ▼                    ▼
           ┌────────────────┐               ┌────────────────┐      ┌────────────────┐
           │  Elasticsearch │               │    SNS/SQS     │      │  Another DB    │
           │  (Search)      │               │  (Notifications)│      │  (Replication) │
           └────────────────┘               └────────────────┘      └────────────────┘
```

## Global Tables

```
┌─────────────────────────────────────────────────────────────┐
│                     Global Tables                            │
├─────────────────────────────────────────────────────────────┤
│  • Multi-region, multi-active replication                   │
│  • Fully managed by AWS                                      │
│  • Sub-second replication latency                           │
│  • 99.999% availability SLA                                 │
│  • Conflict resolution: Last Writer Wins                    │
│  • Requires DynamoDB Streams enabled                        │
│  • Requires same table settings across regions              │
└─────────────────────────────────────────────────────────────┘

                      Global Table
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   ┌─────────┐       ┌─────────┐       ┌─────────┐
   │US-East-1│◄─────►│EU-West-1│◄─────►│AP-Tokyo │
   │ Replica │       │ Replica │       │ Replica │
   └─────────┘       └─────────┘       └─────────┘
        │                  │                  │
        │     Bi-directional Replication      │
        └──────────────────┴──────────────────┘

Use Cases:
• Disaster recovery
• Low-latency global access
• Data locality compliance
```

## DAX (DynamoDB Accelerator)

```
┌─────────────────────────────────────────────────────────────┐
│                          DAX                                 │
├─────────────────────────────────────────────────────────────┤
│  • In-memory cache for DynamoDB                             │
│  • Microsecond response times                               │
│  • Fully managed cluster                                    │
│  • Write-through caching                                    │
│  • API compatible (drop-in replacement)                     │
│  • Eventually consistent reads ONLY                         │
│  • Not suitable for strongly consistent reads               │
└─────────────────────────────────────────────────────────────┘

Performance:
• Without DAX: 1-10 milliseconds
• With DAX: 1-100 microseconds (up to 10x faster)

Architecture:
┌────────────┐     ┌─────────────┐     ┌─────────────┐
│Application │────►│    DAX      │────►│  DynamoDB   │
│            │◄────│   Cluster   │◄────│   Table     │
└────────────┘     └─────────────┘     └─────────────┘
                   (Cache Layer)

Cache Hit: Application ◄── DAX (microseconds)
Cache Miss: Application ◄── DAX ◄── DynamoDB (milliseconds)
```

### When to Use DAX

```
✅ USE DAX FOR:
───────────────
• Read-heavy workloads
• Same items read repeatedly
• Microsecond latency requirements
• Eventually consistent reads acceptable
• Gaming leaderboards
• Social media feeds
• Real-time bidding

❌ DON'T USE DAX FOR:
─────────────────────
• Write-heavy workloads
• Strongly consistent reads required
• Requests are mostly unique (low cache hit)
• Cost-sensitive applications
```

## DynamoDB Transactions

```
┌─────────────────────────────────────────────────────────────┐
│                     Transactions                             │
├─────────────────────────────────────────────────────────────┤
│  • ACID compliant                                           │
│  • All-or-nothing operations                                │
│  • Up to 100 items per transaction                          │
│  • Up to 4 MB total data                                    │
│  • 2x capacity consumption                                   │
│  • Can span multiple tables                                 │
└─────────────────────────────────────────────────────────────┘

Operations:
• TransactWriteItems - Multiple Put/Update/Delete/ConditionCheck
• TransactGetItems - Multiple Get operations
```

### Transaction Example: Bank Transfer

```python
# Transfer $100 from Account A to Account B
client.transact_write_items(
    TransactItems=[
        {
            'Update': {
                'TableName': 'Accounts',
                'Key': {'AccountID': {'S': 'A'}},
                'UpdateExpression': 'SET Balance = Balance - :amount',
                'ConditionExpression': 'Balance >= :amount',
                'ExpressionAttributeValues': {':amount': {'N': '100'}}
            }
        },
        {
            'Update': {
                'TableName': 'Accounts',
                'Key': {'AccountID': {'S': 'B'}},
                'UpdateExpression': 'SET Balance = Balance + :amount',
                'ExpressionAttributeValues': {':amount': {'N': '100'}}
            }
        }
    ]
)
# Both updates succeed or both fail!
```

## TTL (Time To Live)

```
┌─────────────────────────────────────────────────────────────┐
│                    Time To Live (TTL)                        │
├─────────────────────────────────────────────────────────────┤
│  • Automatic item expiration                                │
│  • Specify attribute containing expiration timestamp        │
│  • Unix epoch time in seconds                               │
│  • Items deleted within 48 hours of expiration              │
│  • No additional cost for deletion                          │
│  • Deleted items appear in Streams (if enabled)             │
└─────────────────────────────────────────────────────────────┘

Use Cases:
• Session data
• Temporary tokens
• Log data cleanup
• Event data archival
• Cache invalidation

Example Item:
{
  "SessionID": "sess123",
  "UserID": "user001",
  "TTL": 1735689600  // Unix timestamp for expiration
}
```

---

# 9. Data Modeling

## Single Table Design

```
┌─────────────────────────────────────────────────────────────┐
│                  SINGLE TABLE DESIGN                         │
├─────────────────────────────────────────────────────────────┤
│  • Store multiple entity types in ONE table                 │
│  • Use generic key names (PK, SK)                           │
│  • Overload keys with prefixes                              │
│  • Design for access patterns, not entities                 │
│  • Reduces costs and operational overhead                   │
└─────────────────────────────────────────────────────────────┘

Instead of:
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│  Users  │  │ Orders  │  │Products │  │ Reviews │
└─────────┘  └─────────┘  └─────────┘  └─────────┘

Use:
┌────────────────────────────────────────────────────────────┐
│                     Single Table                            │
└────────────────────────────────────────────────────────────┘
```

### Single Table Example: E-Commerce

```
┌───────────────────┬─────────────────────┬──────────────────────────────┐
│        PK         │         SK          │         Attributes           │
├───────────────────┼─────────────────────┼──────────────────────────────┤
│ USER#user001      │ PROFILE             │ Name, Email, CreatedAt       │
│ USER#user001      │ ORDER#ord001        │ Total, Status, OrderDate     │
│ USER#user001      │ ORDER#ord002        │ Total, Status, OrderDate     │
│ ORDER#ord001      │ PRODUCT#prod001     │ Quantity, Price              │
│ ORDER#ord001      │ PRODUCT#prod002     │ Quantity, Price              │
│ PRODUCT#prod001   │ METADATA            │ Name, Price, Category        │
│ PRODUCT#prod001   │ REVIEW#rev001       │ Rating, Comment, UserID      │
└───────────────────┴─────────────────────┴──────────────────────────────┘

Access Patterns Enabled:
──────────────────────────
1. Get user profile: Query PK="USER#user001", SK="PROFILE"
2. Get user's orders: Query PK="USER#user001", SK begins_with "ORDER#"
3. Get order items: Query PK="ORDER#ord001", SK begins_with "PRODUCT#"
4. Get product reviews: Query PK="PRODUCT#prod001", SK begins_with "REVIEW#"
```

### GSI for Additional Access Patterns

```
GSI1: Inverted Index
─────────────────────
GSI1PK = SK (from base table)
GSI1SK = PK (from base table)

Now you can also query:
• Get all users who ordered product: Query GSI1PK="PRODUCT#prod001"
• Get all orders: Query GSI1PK begins_with "ORDER#"


GSI2: By Status
───────────────
GSI2PK = Status
GSI2SK = OrderDate

Now you can query:
• Get all pending orders: Query GSI2PK="PENDING"
• Get pending orders from last week: Query GSI2PK="PENDING", SK BETWEEN dates
```

## Key Design Patterns

### Hierarchical Data

```
Use sort key for hierarchy:

PK: ORG#amazon
SK: DEPT#engineering
SK: DEPT#engineering#TEAM#backend
SK: DEPT#engineering#TEAM#backend#EMP#john

Query with begins_with:
• Get all in engineering: begins_with("DEPT#engineering")
• Get backend team: begins_with("DEPT#engineering#TEAM#backend")
```

### Many-to-Many Relationships

```
Users ←→ Groups

Base Table:
PK: USER#user1    SK: GROUP#group1
PK: USER#user1    SK: GROUP#group2
PK: GROUP#group1  SK: USER#user1
PK: GROUP#group1  SK: USER#user2

Get user's groups: Query PK="USER#user1", SK begins_with "GROUP#"
Get group's users: Query PK="GROUP#group1", SK begins_with "USER#"
```

### Time-Series Data

```
PK: DEVICE#device001#2024-01
SK: 2024-01-15T10:30:00Z

• Partition by device + month
• Sort by timestamp
• Prevents hot partitions
• Easy to query date ranges
• Easy to archive/delete old data
```

---

# 10. Security

## Encryption

```
┌─────────────────────────────────────────────────────────────┐
│                   Encryption at Rest                         │
├─────────────────────────────────────────────────────────────┤
│  All data encrypted by default (cannot be disabled)         │
│                                                              │
│  Options:                                                    │
│  1. AWS Owned Key (Default)                                 │
│     • Free                                                   │
│     • No management needed                                   │
│                                                              │
│  2. AWS Managed Key (aws/dynamodb)                          │
│     • Visible in KMS                                         │
│     • AWS manages rotation                                   │
│                                                              │
│  3. Customer Managed Key (CMK)                              │
│     • Full control                                           │
│     • Custom rotation                                        │
│     • Audit via CloudTrail                                   │
│     • Additional cost                                        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Encryption in Transit                       │
├─────────────────────────────────────────────────────────────┤
│  • TLS/HTTPS by default                                     │
│  • All API calls encrypted                                  │
│  • VPC endpoints available for private access               │
└─────────────────────────────────────────────────────────────┘
```

## Access Control

### IAM Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/MyTable"
    }
  ]
}
```

### Fine-Grained Access Control

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:Query"],
      "Resource": "arn:aws:dynamodb:*:*:table/Users",
      "Condition": {
        "ForAllValues:StringEquals": {
          "dynamodb:LeadingKeys": ["${aws:userid}"]
        }
      }
    }
  ]
}
// User can only access items where PK = their user ID
```

### VPC Endpoints

```
┌─────────────────────────────────────────────────────────────┐
│                    VPC Endpoint                              │
├─────────────────────────────────────────────────────────────┤
│  • Private connection to DynamoDB                           │
│  • Traffic doesn't leave AWS network                        │
│  • No internet gateway needed                               │
│  • Gateway endpoint (free) or Interface endpoint            │
│  • Enhanced security for sensitive data                     │
└─────────────────────────────────────────────────────────────┘

┌────────────────┐     ┌─────────────────┐     ┌─────────────┐
│   EC2 in VPC   │────►│  VPC Endpoint   │────►│  DynamoDB   │
│                │     │   (Private)     │     │             │
└────────────────┘     └─────────────────┘     └─────────────┘
                              │
                    (No public internet)
```

---

# 11. Performance Optimization

## Avoiding Hot Partitions

```
PROBLEM: Uneven distribution causes throttling

┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   Partition 1: 90% of traffic  ←── HOT PARTITION (Throttled)│
│   Partition 2: 5% of traffic                                │
│   Partition 3: 5% of traffic                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘

SOLUTIONS:
──────────

1. WRITE SHARDING (Add random suffix)
   Before: PK = "2024-01-15"
   After:  PK = "2024-01-15_" + random(1-10)
   
   Distributes writes across 10 partitions

2. COMPOSITE KEYS
   Before: PK = "Status" (only Active/Inactive)
   After:  PK = "Status#" + UserID.substring(0,2)
   
   Creates many more partition values

3. CACHING (DAX)
   - Cache hot items
   - Reduces reads to DynamoDB
   
4. ADAPTIVE CAPACITY
   - Enabled by default
   - Automatically handles some hot partition scenarios
```

## Query Optimization

```
1. USE QUERY INSTEAD OF SCAN
   • Scan reads entire table
   • Query reads only matching partitions
   
2. USE PROJECTION EXPRESSION
   • Return only needed attributes
   • Reduces data transfer
   
3. USE SPARSE INDEXES
   • GSI only contains items with index attributes
   • Smaller index = faster queries
   
4. PARALLEL SCAN (When Scan is needed)
   • Divide table into segments
   • Scan segments in parallel
   • Use for large one-time operations
```

## Batch Operations

```
BatchGetItem:
─────────────
• Read up to 100 items
• Items can be from different tables
• Unprocessed items returned for retry
• Uses eventually consistent by default

BatchWriteItem:
───────────────
• Write/delete up to 25 items
• Items can be from different tables
• No update (only put/delete)
• Unprocessed items returned for retry
• Implement exponential backoff for retries
```

---

# 12. Common Interview Questions

## Q1: When would you choose DynamoDB over RDS?

```
Choose DynamoDB:
────────────────
• Massive scale (millions of requests/sec)
• Simple access patterns (key-based lookups)
• Flexible schema
• Serverless architecture
• Predictable low latency
• Auto-scaling needs

Choose RDS:
───────────
• Complex queries with joins
• ACID transactions across many tables
• Ad-hoc queries
• Fixed schema requirements
• SQL expertise in team
• Complex reporting
```

## Q2: Explain hot partitions and how to prevent them

```
Hot Partition: Single partition receiving disproportionate traffic

Causes:
• Poor partition key design (low cardinality)
• Temporal data without sharding
• Popular items

Prevention:
1. High cardinality partition keys
2. Write sharding (random suffixes)
3. Composite keys
4. Caching with DAX
5. Distribute load across time
```

## Q3: What is Single Table Design and when to use it?

```
Single Table Design:
• Store multiple entity types in one table
• Use generic PK/SK with prefixes
• Design for access patterns

When to use:
• Multiple related entities
• Need to fetch related data in single query
• Want to minimize table count
• Complex relationships

When NOT to use:
• Very simple data model
• Unrelated entities
• Team unfamiliar with pattern
```

## Q4: Explain GSI vs LSI

```
GSI:
• Different partition key allowed
• Eventually consistent only
• Separate throughput
• Can create anytime
• Limit: 20 per table

LSI:
• Same partition key required
• Strongly consistent available
• Shared throughput
• Create at table creation only
• Limit: 5 per table
```

## Q5: How do DynamoDB Streams work?

```
• Capture item-level changes
• Time-ordered log
• 24-hour retention
• Near real-time
• Four view types (keys, new, old, both)
• Integrate with Lambda
• Used for: replication, triggers, analytics
```

## Q6: Calculate RCUs and WCUs

```
RCU Calculation:
• 1 RCU = 1 strongly consistent read up to 4KB
• 1 RCU = 2 eventually consistent reads up to 4KB
• Formula: ceil(item_size / 4KB) × reads/sec

WCU Calculation:
• 1 WCU = 1 write up to 1KB
• Formula: ceil(item_size / 1KB) × writes/sec
```

## Q7: What is DAX and when to use it?

```
DAX = DynamoDB Accelerator (in-memory cache)

Use when:
• Need microsecond latency
• Read-heavy workload
• Same items read repeatedly
• Eventually consistent is acceptable

Don't use when:
• Need strongly consistent reads
• Write-heavy workload
• Low cache hit ratio expected
```

## Q8: Explain DynamoDB transactions

```
• ACID compliant
• Up to 100 items
• Up to 4 MB
• 2x capacity cost
• All-or-nothing
• TransactWriteItems / TransactGetItems
• Use for: financial transfers, inventory updates
```

## Q9: How does Global Tables work?

```
• Multi-region, multi-active
• Automatic replication
• Sub-second latency
• Last Writer Wins conflict resolution
• 99.999% availability
• Requires Streams enabled
• Same table settings all regions
```

## Q10: Explain TTL in DynamoDB

```
• Automatic item expiration
• Specify epoch timestamp attribute
• Deleted within 48 hours
• No cost for deletions
• Shows in Streams if enabled
• Use for: sessions, tokens, logs
```

---

# 13. Code Examples

## Python (boto3)

### Basic Operations

```python
import boto3
from boto3.dynamodb.conditions import Key, Attr
from decimal import Decimal

# Initialize
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('MyTable')

# PutItem
table.put_item(
    Item={
        'PK': 'USER#user001',
        'SK': 'PROFILE',
        'Name': 'John Doe',
        'Email': 'john@example.com',
        'Age': 30
    }
)

# GetItem
response = table.get_item(
    Key={
        'PK': 'USER#user001',
        'SK': 'PROFILE'
    }
)
item = response.get('Item')

# GetItem with strongly consistent read
response = table.get_item(
    Key={
        'PK': 'USER#user001',
        'SK': 'PROFILE'
    },
    ConsistentRead=True
)

# UpdateItem
table.update_item(
    Key={
        'PK': 'USER#user001',
        'SK': 'PROFILE'
    },
    UpdateExpression='SET #n = :name, Age = Age + :inc',
    ExpressionAttributeNames={'#n': 'Name'},
    ExpressionAttributeValues={
        ':name': 'John Smith',
        ':inc': 1
    },
    ReturnValues='UPDATED_NEW'
)

# DeleteItem
table.delete_item(
    Key={
        'PK': 'USER#user001',
        'SK': 'PROFILE'
    }
)
```

### Query Operations

```python
# Query with partition key only
response = table.query(
    KeyConditionExpression=Key('PK').eq('USER#user001')
)
items = response['Items']

# Query with sort key condition
response = table.query(
    KeyConditionExpression=Key('PK').eq('USER#user001') & 
                          Key('SK').begins_with('ORDER#')
)

# Query with filter
response = table.query(
    KeyConditionExpression=Key('PK').eq('USER#user001'),
    FilterExpression=Attr('Status').eq('Active')
)

# Query with projection
response = table.query(
    KeyConditionExpression=Key('PK').eq('USER#user001'),
    ProjectionExpression='PK, SK, #n',
    ExpressionAttributeNames={'#n': 'Name'}
)

# Query GSI
response = table.query(
    IndexName='GSI1',
    KeyConditionExpression=Key('GSI1PK').eq('STATUS#Active')
)

# Query with pagination
response = table.query(
    KeyConditionExpression=Key('PK').eq('USER#user001'),
    Limit=10
)
items = response['Items']

while 'LastEvaluatedKey' in response:
    response = table.query(
        KeyConditionExpression=Key('PK').eq('USER#user001'),
        Limit=10,
        ExclusiveStartKey=response['LastEvaluatedKey']
    )
    items.extend(response['Items'])
```

### Batch Operations

```python
# Batch Write
with table.batch_writer() as batch:
    for i in range(100):
        batch.put_item(
            Item={
                'PK': f'USER#user{i:03d}',
                'SK': 'PROFILE',
                'Name': f'User {i}'
            }
        )

# Batch Get
response = dynamodb.batch_get_item(
    RequestItems={
        'MyTable': {
            'Keys': [
                {'PK': 'USER#user001', 'SK': 'PROFILE'},
                {'PK': 'USER#user002', 'SK': 'PROFILE'},
                {'PK': 'USER#user003', 'SK': 'PROFILE'}
            ]
        }
    }
)
items = response['Responses']['MyTable']
```

### Transactions

```python
client = boto3.client('dynamodb')

# TransactWriteItems
client.transact_write_items(
    TransactItems=[
        {
            'Put': {
                'TableName': 'MyTable',
                'Item': {
                    'PK': {'S': 'ORDER#ord001'},
                    'SK': {'S': 'METADATA'},
                    'Total': {'N': '99.99'}
                }
            }
        },
        {
            'Update': {
                'TableName': 'MyTable',
                'Key': {
                    'PK': {'S': 'USER#user001'},
                    'SK': {'S': 'PROFILE'}
                },
                'UpdateExpression': 'SET OrderCount = OrderCount + :inc',
                'ExpressionAttributeValues': {':inc': {'N': '1'}}
            }
        }
    ]
)

# TransactGetItems
response = client.transact_get_items(
    TransactItems=[
        {
            'Get': {
                'TableName': 'MyTable',
                'Key': {
                    'PK': {'S': 'USER#user001'},
                    'SK': {'S': 'PROFILE'}
                }
            }
        },
        {
            'Get': {
                'TableName': 'MyTable',
                'Key': {
                    'PK': {'S': 'ORDER#ord001'},
                    'SK': {'S': 'METADATA'}
                }
            }
        }
    ]
)
```

### Conditional Operations

```python
# Conditional Put (only if item doesn't exist)
try:
    table.put_item(
        Item={
            'PK': 'USER#user001',
            'SK': 'PROFILE',
            'Name': 'John'
        },
        ConditionExpression='attribute_not_exists(PK)'
    )
except ClientError as e:
    if e.response['Error']['Code'] == 'ConditionalCheckFailedException':
        print('Item already exists')

# Optimistic Locking
table.update_item(
    Key={
        'PK': 'USER#user001',
        'SK': 'PROFILE'
    },
    UpdateExpression='SET #n = :name, Version = :newVersion',
    ConditionExpression='Version = :expectedVersion',
    ExpressionAttributeNames={'#n': 'Name'},
    ExpressionAttributeValues={
        ':name': 'John Updated',
        ':newVersion': 2,
        ':expectedVersion': 1
    }
)
```

---

# 14. Quick Reference

## Limits

| Limit | Value |
|-------|-------|
| Item size | 400 KB |
| Partition key length | 2048 bytes |
| Sort key length | 1024 bytes |
| Attribute name length | 64 KB |
| Table name length | 3-255 characters |
| GSIs per table | 20 |
| LSIs per table | 5 |
| Projected attributes per index | 100 |
| Partition throughput | 3000 RCU, 1000 WCU |
| BatchGetItem | 100 items, 16 MB |
| BatchWriteItem | 25 items, 16 MB |
| TransactGetItems | 100 items, 4 MB |
| TransactWriteItems | 100 items, 4 MB |
| Query/Scan result | 1 MB |

## Pricing Components

```
1. Capacity
   ─────────
   • On-Demand: Per request pricing
   • Provisioned: Per hour based on RCU/WCU
   
2. Storage
   ────────
   • $0.25 per GB-month
   
3. Optional Features
   ──────────────────
   • DAX: Per node-hour
   • Streams: Per 100K read requests
   • Global Tables: Per replicated WCU
   • Backup: Per GB-month
   • Restore: Per GB restored
   • Data Transfer: Per GB (out)
```

## Key Formulas

```
RCU (Strongly Consistent):
  RCUs = ⌈ItemSize / 4KB⌉ × ReadsPerSecond

RCU (Eventually Consistent):
  RCUs = ⌈ItemSize / 4KB⌉ × ReadsPerSecond / 2

RCU (Transactional):
  RCUs = ⌈ItemSize / 4KB⌉ × ReadsPerSecond × 2

WCU (Standard):
  WCUs = ⌈ItemSize / 1KB⌉ × WritesPerSecond

WCU (Transactional):
  WCUs = ⌈ItemSize / 1KB⌉ × WritesPerSecond × 2
```

## CLI Commands

```bash
# Create table
aws dynamodb create-table \
  --table-name MyTable \
  --attribute-definitions AttributeName=PK,AttributeType=S AttributeName=SK,AttributeType=S \
  --key-schema AttributeName=PK,KeyType=HASH AttributeName=SK,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST

# Put item
aws dynamodb put-item \
  --table-name MyTable \
  --item '{"PK": {"S": "user001"}, "SK": {"S": "profile"}, "Name": {"S": "John"}}'

# Get item
aws dynamodb get-item \
  --table-name MyTable \
  --key '{"PK": {"S": "user001"}, "SK": {"S": "profile"}}'

# Query
aws dynamodb query \
  --table-name MyTable \
  --key-condition-expression "PK = :pk" \
  --expression-attribute-values '{":pk": {"S": "user001"}}'

# Scan
aws dynamodb scan --table-name MyTable

# Delete table
aws dynamodb delete-table --table-name MyTable
```

---

# Summary Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     DynamoDB Quick Reference                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  KEYS:                                                                   │
│  • Partition Key (PK) - Required, determines partition                   │
│  • Sort Key (SK) - Optional, enables range queries                       │
│                                                                          │
│  INDEXES:                                                                │
│  • GSI - Different PK/SK, eventually consistent, separate throughput    │
│  • LSI - Same PK, different SK, strongly consistent, shared throughput  │
│                                                                          │
│  CAPACITY:                                                               │
│  • 1 RCU = 4KB strongly consistent read                                 │
│  • 1 WCU = 1KB write                                                    │
│  • On-Demand = Pay per request, no planning                             │
│  • Provisioned = Set RCU/WCU, cheaper for steady workloads             │
│                                                                          │
│  CONSISTENCY:                                                            │
│  • Eventually (default) - 0.5 RCU, may be stale                         │
│  • Strongly - 1 RCU, always latest                                      │
│  • Transactional - 2 RCU, ACID compliant                                │
│                                                                          │
│  OPERATIONS:                                                             │
│  • Query - Efficient, uses PK + optional SK                             │
│  • Scan - Reads entire table, expensive                                 │
│  • BatchGet/Write - Multiple items, 100/25 limit                        │
│  • Transactions - ACID, 100 items, 4MB, 2x cost                         │
│                                                                          │
│  ADVANCED:                                                               │
│  • Streams - Change capture, 24hr retention, Lambda integration         │
│  • Global Tables - Multi-region, multi-active replication               │
│  • DAX - In-memory cache, microsecond latency                           │
│  • TTL - Automatic expiration, no cost                                  │
│                                                                          │
│  BEST PRACTICES:                                                         │
│  • High cardinality partition keys                                      │
│  • Single table design for related entities                             │
│  • Query > Scan                                                          │
│  • Use GSI for additional access patterns                               │
│  • Cache with DAX for read-heavy workloads                              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

*Good luck with your DynamoDB interview!*
