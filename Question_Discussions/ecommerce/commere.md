# Amazon Retail System Design – README

Welcome to the **Amazon Retail System Design**! This document presents a comprehensive, end-to-end architecture for a scalable, high-performance e-commerce platform inspired by Amazon (or Flipkart). The design focuses on product listings, cart service, inventory management, order processing, and optimizing read throughput.

---

## 🔍 Overview

We're designing a retail platform where users can:

* View products with prices and availability
* Add products to a cart
* Place an order

The system must be scalable, resilient, and highly responsive to ensure conversions. It is optimized to handle **1 billion products**, **10 million orders per day**, and maintain **low latency reads**.

---

## 📊 Capacity Estimates

* Products: 1 billion (\~1 MB each) → \~1 PB of product data
* Orders: 10M/day = \~100 orders/sec
* Images served via S3 + CDN

---

## 📘 Components Overview

### 🛏️ Product Database

* **MongoDB** for flexible schema (JSON per product)
* Partitioned by product ID
* Uses **B-tree indexes** for fast reads
* Cached popular products in **Redis**
* Product images served via **CDN**

### 🛒 Cart Service

* Cart modeled as a **CRDT Observe-Remove Set**
* Each cart action gets a unique tag (UUID)
* Stored in **Riak** or similar CRDT-supporting DB
* Partitioned by cart ID
* Ensures eventual consistency across nodes

### 📝 Order Service

* Orders flow through a **Kafka queue** (ensures atomicity)
* **Flink** used to split and route product-level orders
* Inventory tracked per product via stateful Flink nodes
* Writes inventory updates to MongoDB
* Sends confirmation emails (requires idempotent design or 2PC)

### ⚖️ Inventory Updates

* Seller inventory changes written to Kafka
* Ingested by Flink nodes
* Updated stock reflected in-memory and persisted

### 📈 Popularity Tracking

* **Spark Streaming** aggregates product popularity daily
* Metrics exported to **HDFS**
* **Batch Spark Job** ranks products
* Popular products preloaded into Redis and indexed in Elasticsearch

---

## 🧰 Search & Indexing

* **Elasticsearch** stores product search index
* Uses **local indexing** per node
* Partitioned by:

  * Product category
  * Popularity score range (for pagination & balance)
* Stores mini product descriptions to avoid extra DB reads

---

## ⚖️ Read Optimization

* Popular items cached in Redis
* Images served by CDN for locality
* Standby clusters populated before promotion
* Cluster switch coordinated via **Zookeeper**

---

## ⚡ Final Flow Summary

1. **Search** term hits Elasticsearch (partitioned by category + popularity)
2. **Product Click** loads from Redis or MongoDB
3. **Add to Cart** updates CRDT (Riak)
4. **Submit Order** goes to Kafka queue
5. **Flink** routes and validates per-product order
6. **Inventory** decremented in-memory & persisted
7. **Popularity** tracked via Spark -> HDFS
8. **Reindexing** & **cache refresh** in standby clusters

---

## 🚀 Technologies Used

| Component       | Tech Stack             |
| --------------- | ---------------------- |
| Product DB      | MongoDB + Redis        |
| Cart            | Riak (CRDT Set)        |
| Orders          | Kafka + Flink          |
| Inventory       | Flink + Kafka          |
| Popularity      | Spark Streaming + HDFS |
| Search          | Elasticsearch          |
| Image Hosting   | S3 + CDN               |
| Cluster Control | Zookeeper + DNS        |

---

## 💪 Resilience & Scalability

* All write paths are **idempotent**
* Kafka ensures **message durability**
* Flink checkpoints enable **stream recovery**
* Redis caching for **high-QPS reads**
* CDN for **fast, distributed content serving**

---

## 🚀 Final Notes

This system design balances **flexibility**, **scalability**, and **eventual consistency** to create a retail experience that is both performant and resilient. Perfect for system design interviews or real-world architectural inspiration.

---

Happy designing! 🚀
