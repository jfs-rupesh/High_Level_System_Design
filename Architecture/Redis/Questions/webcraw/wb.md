![ System Architecture](../diagrams/craw.png)


## 🔍 What Is Web Crawling?

Web crawling is the process of systematically browsing the web to download and index content. Applications include:

* Building search engines (e.g., Google)
* Collecting data for training language models (e.g., OpenAI)
* Monitoring and compliance (e.g., content validation, SEO tools)

---

## ⚡ Project Objectives

1. **Crawl** all publicly accessible web pages
2. **Store** scraped content efficiently
3. **Respect** `robots.txt` directives
4. **Complete** the crawl in under 1 week

---

## 📊 Capacity Planning

* **1 billion pages x \~1MB** per page = \~1PB of raw data
* 1 week = \~600,000 seconds
* 1.5K requests/sec required
* Rough estimate: **100 servers** needed, assuming multithreaded processors

---

## 🌐 Core Processing Pipeline

1. Pull URL from frontier
2. Check if URL has already been crawled
3. Validate with `robots.txt`
4. Resolve DNS to get IP
5. Make HTTP request and fetch content
6. Check for duplicate content using content hash
7. Parse and store content (e.g., to S3 or HDFS)
8. Extract new URLs and add to the frontier

---

## 🏋️ Design Optimizations

### URL Frontier Management

* Use **Kafka + Flink** for frontier coordination and task distribution
* Host-based partitioning for cache locality and crawl policy reuse
* **Breadth-first traversal** with queues (not stacks)

### Deduplication

* Use **Redis** or **CRDT** sets for fast hash lookups
* Content hash used to avoid re-storing identical HTML

### Caching

* DNS & `robots.txt` results cached per node
* LRU eviction strategy for stale host entries

### Fault Tolerance

* Use **Flink's checkpointing** and Kafka's **message persistence** for recoverability
* All external interactions must be **idempotent** (e.g., S3 writes by content hash)

---

## 📚 Technology Stack

| Component         | Technology      | Purpose                          |
| ----------------- | --------------- | -------------------------------- |
| Message Queue     | Kafka           | URL frontier distribution        |
| Stream Processing | Flink           | Caching, stateful URL processing |
| Hash Storage      | Redis / CRDT    | Duplicate content detection      |
| DNS Resolution    | System Resolver | Host-to-IP translation           |
| Object Storage    | AWS S3          | Persistent content store         |
| Load Balancer     | Custom / DNS LB | Even task distribution           |

---

## 🚀 Deployment Tips

* Co-locate Flink + Kafka in same region or data center for low-latency
* Match S3 region with EC2 processing region
* Horizontally scale processing nodes
* Ensure Flink can re-ingest from Kafka offset checkpoints

---

## 🔧 Future Improvements

* Add priority queue for high-quality sites
* Integrate natural language filter before storage
* Evaluate benefits of geo-based crawl partitioning
* Add web UI for monitoring crawl progress

---

## 🙋 Contributors

This README was generated based on a detailed walkthrough and system design explanation of a high-performance distributed web crawler. Huge thanks to the original speaker and contributors who outlined this architecture.

---

Happy crawling! 🚀
