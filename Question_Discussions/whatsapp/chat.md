Here’s a comprehensive and well-organized **README.md** file for your **Facebook Messenger / WhatsApp-style messaging system**, based on your detailed walkthrough:

---

# 📨 Facebook Messenger / WhatsApp System Design

A high-level design and architecture for building a **real-time, scalable, distributed messaging service** similar to **Facebook Messenger** or **WhatsApp**. This service supports:

* Real-time chat
* Group messaging (up to 10 users)
* Persistent history across devices
* Device fanout via WebSockets
* Consistent message delivery and ordering

---

## 🧠 Summary

| Feature                    | Description                                  |
| -------------------------- | -------------------------------------------- |
| ✅ Group Chats              | Up to 10 users per chat (configurable)       |
| 📥 Message Persistence     | Stored in HBase with ordering and metadata   |
| 📤 Fanout Architecture     | Kafka → Flink → Chat Servers                 |
| 📶 Real-time Delivery      | WebSockets per device                        |
| 💾 Historical Read Support | Load old messages by chat ID                 |
| ♻️ Resilience              | Uses Kafka, idempotent writes, and Zookeeper |

---

## 📏 Problem Requirements

* Users must be able to **send** and **receive** messages in real time
* Chats can include **up to 10 participants**
* Messages must be **persisted** for device sync and history
* Messages should be delivered to **all active client devices**
* Clients should be able to **load previous conversations**

---

## 📐 Capacity Estimates

| Metric             | Estimate  |
| ------------------ | --------- |
| Users              | 1 billion |
| Messages/user/day  | 100       |
| Message size (avg) | 100 bytes |
| Daily storage      | \~10 TB   |
| Yearly storage     | \~4 PB    |

---

## 🧱 Core Database Schemas

### 1. `ChatMembers` Table (MySQL)

Used to find which chats a user is part of.

```sql
CREATE TABLE chat_members (
  user_id BIGINT,
  chat_id BIGINT,
  PRIMARY KEY (user_id, chat_id)
);
```

* 🔑 Partitioned & indexed by `user_id` for fast lookup
* 🛡 Single-leader replication ensures **causal consistency**

---

### 2. `Users` Table (MySQL)

Basic user data for login & metadata.

```sql
CREATE TABLE users (
  user_id BIGINT PRIMARY KEY,
  email VARCHAR(255),
  password_hash TEXT
);
```

* 📌 Partitioned on `user_id`
* ⚖ Load-balanced via consistent hashing

---

### 3. `Messages` Table (HBase)

Stores all message history per chat.

```
Row Key: <chat_id + timestamp + uuid>
Columns: sender_id, content, metadata, timestamp
```

* 🗃 Partitioned by `chat_id`
* ⏱ Sorted by `timestamp` and `uuid` for **ordering + idempotency**
* 📊 Column-oriented for fast selective reads (skip unused metadata)

---

## 🔄 Message Delivery Flow

### 1. **Client Sends Message**

* Hits `WebSocket` → `ChatService`
* ChatService publishes message to **Kafka**, partitioned by `chat_id`

### 2. **Kafka → Flink**

* Flink consumes from Kafka
* Maintains in-memory cache of chat members via CDC on `chat_members`
* Broadcasts messages to chat participants’ chat servers

### 3. **Flink → Chat Servers**

* Uses **load balancer + Zookeeper** to route by user ID
* Fanout to devices through active WebSocket connections

### 4. **Message Persistence**

* Flink also writes message to HBase (idempotent upsert via `uuid`)

---

## 🕸 Real-Time Protocol

### ✅ WebSockets

* Chosen for **bi-directional**, persistent, low-overhead connections
* Handles both incoming and outgoing messages per user session

### ❌ Alternatives (Not Used)

| Option             | Why Not?                                  |
| ------------------ | ----------------------------------------- |
| Long Polling       | Too much overhead, frequent reconnections |
| Server-Sent Events | One-way only (server → client)            |

---

## 🧮 Load Balancing Strategy

* **Consistent Hashing** used to assign users to `ChatService` instances
* **Zookeeper** stores and updates the hash ring
* Load balancers (in active-active config) listen to Zookeeper

### Reconnects:

* Clients periodically send heartbeats
* On timeout or server crash:

  * Re-resolve server via load balancer
  * Add jitter to avoid thundering herd

---

## 💡 CDC: Change Data Capture

Used to sync:

* `chat_members` updates to Flink
* Enable **local caching** of group participants per chat
* Reduces DB load by avoiding repetitive reads

---

## 🧪 Message Idempotency

Each message is assigned a **UUID** on the server:

* Stored with timestamp in HBase
* Enables **upserts**, avoiding duplicate inserts during retries

---

## 🛠 Tech Stack

| Component          | Technology                    | Reasoning                           |
| ------------------ | ----------------------------- | ----------------------------------- |
| Messaging Broker   | Kafka                         | Partitioned, durable log            |
| Stream Processor   | Apache Flink                  | Real-time fanout, CDC ingestion     |
| Storage (Users)    | MySQL                         | Consistency & simple queries        |
| Storage (Messages) | HBase                         | Write-heavy, columnar reads         |
| Cache / Lookup     | Flink local state + Zookeeper | Fast access to group/chat mappings  |
| Real-time Protocol | WebSocket                     | Persistent bi-directional messaging |

---

## 🎯 Summary Architecture Diagram

```plaintext
Client → Load Balancer → Chat Service (WebSocket)
       ↘                        ↗
        Kafka ← Chat Service ← Flink ← CDC (chat_members)
             ↘              ↘
              HBase (Messages)   Active Client Devices
```

---

## 📚 Future Improvements

* Message reactions, deletes, and edits
* Push notifications for offline devices
* Multi-device sync status (e.g., read receipts)
* Full end-to-end encryption layer
* Offline sync using CRDTs

---

## 👋 Conclusion

This architecture provides a **resilient**, **real-time**, and **scalable messaging experience** using modern event-driven techniques like CDC, Kafka, and WebSockets.
Built for billions of users, it handles fanout, persistence, and recovery while remaining user-centric and efficient.


