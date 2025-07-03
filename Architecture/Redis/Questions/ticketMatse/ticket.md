# TicketMaster System Design README

Welcome to the system design breakdown for a **TicketMaster-style ticket booking platform**. This README provides an in-depth explanation of the system's architecture, design decisions, and challenges associated with designing a highly available, fair, and scalable online ticketing system.

---

## 🎯 Overview

TicketMaster is a platform where users can:

* Browse events
* View available seats
* Claim tickets for a short duration (e.g., 5 mins)
* Complete a purchase within the claim window

The design must handle:

* High read/write load for popular events
* Race conditions for simultaneous claims
* Fairness (FIFO vs. free-for-all)
* Eventual persistence and fault tolerance

---

## ✅ Functional Requirements

1. Users can view available seats for live events.
2. Seats can be **claimed** for a short time window (e.g., 5 minutes).
3. Once claimed, only the user who claimed the seat can purchase it.
4. If the claim expires or is abandoned, the seat becomes available again.
5. Support for waitlisting or FIFO ordering for fairness.
6. Fault-tolerant booking that ensures consistency.

---

## 🧰 Non-Functional Requirements

* High availability and low latency
* Resilience to race conditions and concurrency
* Ability to scale with traffic spikes (e.g., Taylor Swift concert)
* Consistency guarantees (eventual consistency OK for historical data)
* Efficient memory use for hot data; cold data to disk

---

## ⚙️ Core Concepts & Design Elements

### ⚪ Ticket Claim Model

* Each seat claim includes:

  * `event_id`, `section_id`, `row_id`, `seat_id`
  * `claim_user_id`, `claim_expiry_timestamp`
  * `is_booked`

* Claims are atomic and serializable (no partial seat claims)

### ⚖️ Transaction Strategy

* Use **SQL databases** with transactions for claim/write logic
* Partition tables by `event_id` + `section_id` + `row_id`
* Hot rows can be moved to **dedicated in-memory cache nodes**

---

## 🔄 Claim Expiration and Booking

### ⏱ Claim Expiry

* Claims are valid for 5 minutes
* Timestamp generated **server-side** to avoid client manipulation
* Implemented via expiration timestamp column

### ⚖️ Fencing Token

* Claim = `(user_id, expiry_timestamp)` tuple
* Must be sent with the booking request
* Prevents re-use or replay of stale claims

---

## 🏛️ Fairness Strategies

### Option 1: Free-For-All (Unfair)

* Race to claim open seats once expired
* Leads to thundering herd problems

### Option 2: FIFO Waitlist (Fair)

* Maintain a queue of users per row/section
* Grant seat to next in line when claim expires

---

## 📝 Data Structures Used

### → Doubly Linked List + HashMap

* Linked List: Order of waiting users
* HashMap: O(1) access/removal of any user
* Handles user cancellation/removal from queue efficiently

### 🔬 Expiration Handling

* TreeSet: Index orders by size (number of seats requested)
* As seats are booked or freed, remove invalid (unfillable) orders
* Can be done by scanning the TreeSet and using HashMap for O(1) removal from list

---

## 🚚 Scalability & Partitioning

### Partition by:

* Event ID + Section ID + Row ID for claim server
* User ID for historical booking storage

### 📊 Cache Layer for Hot Rows

* For popular events, store seat state and waitlist **in memory**
* Use custom in-memory service with lock support instead of Redis (to avoid single-threaded limitation)

---

## 🚀 Booking Completion & Fault Tolerance

### Booking Service (Async Processing)

* Uses **Change Data Capture (CDC)** to sync confirmed bookings to:

  * Kafka queue
  * Historical booking DB (e.g., PostgreSQL/MySQL)

### 🚫 Avoiding Two-Phase Commits

* Booking handler updates in-memory data structure (source of truth)
* Persist via CDC to Kafka + DB
* Use write-ahead logs and async replication for durability

---

## 🔍 Client Interaction & Real-Time Feedback

* Users are routed to appropriate partitioned order server
* WebSocket connections allow real-time queue position updates
* WebSocket also used for notification when claim is accepted/expired

---

## 📈 Performance Optimizations

* Consistent hashing for partitions
* Replica servers to handle read-heavy workloads (e.g., booking history)
* Asynchronous background processing for bookings and claims
* Caching hot paths (popular events/rows) in memory

---

## 📊 Final Component Diagram (Described)

* **Client** → Load Balancer → Partitioned Claim Server
* **Claim Server**:

  * Maintains: seat status, waitlist (linked list + hashmap)
  * Replicates state to: backup server
  * Async CDC to: Kafka
* **Kafka** → Consumer → Bookings DB (partitioned by user\_id)
* **Client** fetches bookings via Bookings Service

---

## 📄 Summary

This design focuses on high-throughput, fair ticket booking with support for waitlists, expiration logic, and fault-tolerant booking. It trades strong consistency in some places for performance and user experience, while still being robust enough to recover from faults.

---

