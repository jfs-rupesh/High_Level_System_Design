Sure! Here's a detailed and organized `README.md` file based on the full walkthrough of your **Distributed Job Scheduler** design:

---

# 🛠️ Distributed Job Scheduler

Welcome to the **Distributed Job Scheduler** — a scalable, fault-tolerant system that enables execution of uploaded binaries (jobs) via CRON schedules or DAG workflows. Designed to run millions of jobs daily across distributed executors.

---

## 📌 Table of Contents

* [What is a Job Scheduler](#what-is-a-job-scheduler)
* [Problem Requirements](#problem-requirements)
* [System Overview](#system-overview)
* [Task Scheduling](#task-scheduling)

  * [CRON Scheduling](#cron-scheduling)
  * [DAG Scheduling](#dag-scheduling)
* [Scheduler Table Design](#scheduler-table-design)
* [Load Balancing & Task Execution](#load-balancing--task-execution)
* [Job Status & History](#job-status--history)
* [Exactly Once Semantics & Locking](#exactly-once-semantics--locking)
* [Full Architecture Diagram](#full-architecture-diagram)
* [Technologies Used](#technologies-used)

---

## ❓ What is a Job Scheduler

At its core, a job scheduler:

* Accepts **binaries (e.g. JARs, scripts)** uploaded by users.
* Executes them based on a **CRON schedule** or **DAG dependencies**.
* Supports **fault tolerance, retries, and observability**.
* Maintains **only-once** execution semantics (as best-effort).

---

## 📋 Problem Requirements

* ✅ Run uploaded binaries on schedule or via DAG logic.
* ✅ Every job should ideally run **once**.
* ✅ Expose **status/history** to users.
* ✅ Support both **CRON** and **DAG** task structures.
* ✅ Handle **millions of jobs/day** efficiently.

---

## 🧠 System Overview

The architecture consists of the following:

* **CRON Table** → Stores schedules.
* **DAG Table** → Stores task dependencies.
* **Scheduler Table** → Main queue of tasks.
* **Executors** → Workers that download and run jobs.
* **Status Table** → Job completion state.
* **ActiveMQ (or in-memory queues)** → Balancing workloads.

Jobs flow like this:

1. Jobs uploaded → stored in **S3**.
2. Metadata recorded in **DAG/CRON tables**.
3. Scheduled into **scheduler table** (via Change Data Capture or direct).
4. Executor nodes **poll, run**, and **update** statuses.
5. DAGs use **epoch tracking** to determine next job.

---

## 🕐 CRON Scheduling

### Tables:

* `cron_jobs`: Stores recurrence info (e.g. every Mon @ 3PM).
* `scheduled_tasks`: Only contains the **next upcoming instance**.

### Flow:

1. On job creation, insert to both tables.
2. Executor picks up from `scheduled_tasks`, runs job.
3. After success, re-schedule **next run** by updating `scheduled_tasks`.

---

## 🔗 DAG Scheduling

### DAG Table Schema:

| job\_id | cron? | children | dependencies | epoch\_data |
| ------- | ----- | -------- | ------------ | ----------- |

### Flow:

1. Initial CRON roots scheduled directly.
2. When root completes, mark **dependent nodes** with updated epoch.
3. If all dependencies are on the same epoch → schedule the node.
4. Leaf completion triggers re-scheduling of the root (cycle).

### Key Concepts:

* Epoch tracking = "wave" of DAG execution.
* Avoids overlapping execution.
* All updates use **atomic multi-row transactions** or are confined to single-partition DAGs.

---

## 🧾 Scheduler Table Design

| job\_id | s3\_url | run\_timestamp | status |
| ------- | ------- | -------------- | ------ |

### Features:

* ✅ Indexed by `run_timestamp` to support polling.
* ✅ Updated **after assignment** to reflect timeout/retry windows.
* ✅ Used by executors to determine eligibility.
* ✅ Includes `status` column (`IN_PROGRESS`, `COMPLETED`, `FAILED`).

### Optimization:

* Use **partitioned replicas**.
* Executors **only poll jobs** where `status != COMPLETED/FAILED AND run_timestamp <= now`.

---

## ⚖️ Load Balancing & Task Execution

### Option 1: Naive Load Balancer

* Not ideal — unaware of executor load/state.

### ✅ Option 2: In-Memory Message Queues (e.g. ActiveMQ)

* Executors pull tasks only **when idle**.
* Efficient use of resources.
* Built-in retry logic.

### 🔁 Multi-Level Queues (for heterogenous hardware):

* Level 1: Fast/lightweight jobs (short timeout).
* Level 2: Medium jobs.
* Level 3: Long-running/heavy jobs.
* Higher-tier executors **read lower queues too** if idle.

---

## 📊 Job Status & History

* Users query **read-only replica** of `scheduler table` for status.
* Job completion = updates status field.
* Prevents unnecessary retries.
* Allows debugging (e.g. error messages/logs stored in S3 or log service).

---

## ✅ Exactly Once Semantics & Locking

> Running once is *hard* — here’s how we mitigate:

### Problems:

* Executor crashes → duplicate runs.
* Timeout → job rescheduled while still running.
* ActiveMQ doesn't ACK in time → job redelivered.

### Solutions:

* Use **Zookeeper distributed locks**:

  * Lock per job ID before execution.
  * TTL to prevent deadlocks.
* **Make jobs idempotent** when possible.
* Add **item potency key** if job writes to external state (e.g. database).

---

## 🗺️ Full Architecture Diagram

```plaintext
     +-------------+          +-------------------+
     |  User/API   | ───────> | DAG / CRON Tables |
     +-------------+          +-------------------+
                                   │
        Change Data Capture        ▼
                              +------------------+
                              | Scheduler Table  |
                              +------------------+
                                   │
                              (polling every X sec)
                                   ▼
                          +------------------------+
                          | ActiveMQ / In-Mem Queues|<──────┐
                          +------------------------+       │
                                   │                       ▼
                         +------------------+       +------------------+
                         | Executor Nodes   |──────▶| Job Status Update|
                         +------------------+       +------------------+
                                   │
                            Runs Job from S3
```

---

## 🔧 Technologies Used

| Component           | Tech                       |
| ------------------- | -------------------------- |
| Binary Storage      | AWS S3 or MinIO            |
| Scheduler DB        | MySQL / Postgres / MongoDB |
| DAG Storage         | MongoDB (flexible schema)  |
| Queues              | ActiveMQ / RabbitMQ        |
| Distributed Locking | Apache ZooKeeper           |
| CDC & Streams       | Kafka + Flink (optional)   |
| Status Storage      | Read-only DB replicas      |

---

## ✅ Best Practices

* Use **Change Data Capture (CDC)** over 2PC when possible.
* Keep **scheduler polling lightweight**.
* Use **Bloom filters or job\_id indexes** to deduplicate retry attempts.
* Shard jobs across partitions by time/job\_id hash.
* Ensure DAGs fit within a single DB partition.

---


