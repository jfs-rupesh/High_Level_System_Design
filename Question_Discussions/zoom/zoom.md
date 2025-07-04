Here's a professional and comprehensive `README.md` based on your transcript-style content. This README is structured to reflect the video discussion, summarizing the system design of a **Video Group Chat Application** similar to Zoom or Skype.

---

# 📹 Video Group Chat System - System Design Overview

Welcome to the system design of a **Video Group Chat Application** akin to Zoom or Skype. This document captures the key architectural components, networking principles, scaling strategies, and storage mechanisms involved in building a robust, scalable group video chat service.

> *Note: This project originated from a conceptual walkthrough recorded during a weekday evening grind session before the Indy 500 trip. If you're reading this on a Sunday... the narrator might be in a field somewhere...*

---

## 🚀 Features and Requirements

### Functional Requirements:

1. Support for **1:1 and group video chats**.
2. Support **up to 100 users per call**.
3. Enable **server-side recording** of video calls for later access.

### Non-Functional Requirements:

* Low latency.
* Scalable to support large concurrent user bases.
* Fault-tolerant with high availability.
* Efficient media handling and network usage.

---

## 🌐 Networking Protocols

* **UDP for media (video/audio)**: Prioritizes speed over reliability. Dropped packets (frames) are tolerable.
* **TCP for signaling/control**: Reliable messaging used for metadata and connection setup.

---

## 🔄 Peer-to-Peer (P2P) vs Centralized Model

### One-on-One Calls:

* **Peer-to-peer communication** is ideal.
* Uses **STUN servers** for NAT traversal and IP discovery.
* May fall back to relayed connections if NAT type blocks P2P.

### Group Calls:

* **Centralized Media Server (SFU - Selective Forwarding Unit)** approach is preferred.
* Reduces client load by handling video/audio streams centrally.

---

## 📡 Selective Forwarding (SFU)

### Key Concept:

* Clients send multiple streams (e.g., 1080p, 720p, 480p).
* Server **forwards only relevant streams** based on client preferences.
* Supports **dynamic layouts** (e.g., HD stream for speaker, SD for others).

### Benefits:

* Reduced bandwidth.
* Customizable user views.
* Lower CPU usage on the server than full transcoding.

---

## 🧠 Media Processing Options

| Strategy                        | Description                              | Pros             | Cons              |
| ------------------------------- | ---------------------------------------- | ---------------- | ----------------- |
| Server Transcoding              | Server converts one stream into multiple | Low client load  | High server CPU   |
| Proxy Transcoding               | Intermediate node transcodes             | Balanced load    | Increased latency |
| Client Multi-Stream (Preferred) | Client sends multiple resolutions        | Best performance | More client load  |

---

## 🧰 WebRTC Abstraction

* **WebRTC** is used as the underlying technology.
* Handles NAT traversal, peer connections, media capture, and transmission.

---

## ⚙️ Partitioning & Load Distribution

### Sharding Strategy:

* Shard calls based on `chatID` using **consistent hashing**.
* Each server handles a subset of calls.

### Horizontal Scaling:

* For heavy calls, **distribute user streams across multiple SFUs**.
* Each client connects to 2-3 servers for read scalability.

---

## 🛡️ Replication & Failover

* Use **Zookeeper** to monitor health of chat servers.

* Employ **active-passive** failover:

  * Passive server monitors the primary.
  * On failure, it becomes active and clients reconnect.

* Stateless architecture: most state re-established via WebSocket metadata.

---

## 🎥 Call Recording Architecture

### Scenario 1: Single Output Recording

* A **recording server subscribes** to streams.
* Encodes the layout (e.g., speaker view) and stores in S3.

### Scenario 2: All Streams Recorded

* **Multiple recording nodes** subscribe to individual user streams.
* Frames are written to **Kafka** (partitioned by `chatID`).
* A **stateful stream processor** consumes, aligns via timestamps, and writes to S3.

---

## 📊 High-Level Architecture Diagram

```
Client
  │
  ├──(TCP)──▶ Load Balancer ──▶ Chat Server (via Consistent Hashing on chatID)
  │                          ├──▶ WebSocket (selective forwarding control)
  │                          └──▶ UDP (video/audio streams at multiple resolutions)
  │
  ├──▶ Passive Chat Server (hot standby)
  │
  └──▶ Recording Servers ──▶ Kafka ──▶ Stateful Consumer ──▶ S3 (recorded footage)
```

---

## 📝 Final Notes

* This design prioritizes **scalability**, **customizability**, and **efficiency**.
* While some areas (e.g., codecs, buffer management) are simplified, this design forms a strong foundation for real-world implementation.

---

## 🏁 Special Thanks

Thanks for reading. If you’re feeling as tired as I was while recording this — congrats, you’re ready for a real systems interview. Enjoy your weekend!

---

A **STUN server** stands for **Session Traversal Utilities for NAT**. It's a network protocol used in NAT (Network Address Translation) traversal — especially important in peer-to-peer (P2P) communications like video calls, VoIP, and online gaming.

### 🧠 What Does It Do?

A **STUN server helps a device discover its public IP address and the type of NAT it's behind**, so it can try to establish a direct connection with another device on the internet.

### 📦 Why It's Needed

Most devices are behind routers or firewalls (NATs), which hide their internal/private IP addresses. But when you want to do peer-to-peer communication (like in a 1-on-1 Zoom call), each peer needs to know:

* What their **public IP and port** are (as seen by the outside world)
* How to **punch through NATs** to reach the other peer

### 🔄 How It Works (Simplified Flow)

1. **Your client contacts the STUN server**.
2. The STUN server replies with the **public IP address and port** it sees.
3. Your client now knows its "public-facing" address.
4. Both clients can now **attempt a direct peer-to-peer connection** using that info.

### 🌐 Example in a Video Chat

Let’s say you and a friend are starting a Zoom call:

* Zoom clients contact a STUN server (Zoom-operated or public).
* Each learns their public IP/port.
* They try to connect directly using UDP.
* If successful, you get **low-latency P2P video**.
* If not (e.g. strict NATs), fallback is used (e.g. TURN or a relay server).

### 🆚 STUN vs TURN

* **STUN** = Discovery. Fast. Lightweight. Helps set up P2P if NAT allows.
* **TURN** = Relay. Used **only if STUN fails**. Slower, but reliable (acts as a middleman).

### ✅ Summary

| Feature      | STUN                                |
| ------------ | ----------------------------------- |
| Stands for   | Session Traversal Utilities for NAT |
| Purpose      | Discovers public IP/port            |
| Used for     | NAT traversal in P2P communication  |
| Lightweight? | Yes                                 |
| Protocols    | Usually uses UDP                    |

---
**WebRTC** stands for **Web Real-Time Communication**. It's an **open-source project and set of protocols/APIs** that enable **real-time audio, video, and data communication** between browsers and devices — **without requiring plugins or third-party software**.

---

### 🧠 In Simple Terms

**WebRTC lets you build Zoom-like apps** directly in the browser.

It handles:

* 📹 Video and 🎙️ audio calls
* 📁 Real-time file/data transfer
* 🔄 Peer-to-peer connections
* 🌐 NAT traversal (via STUN/TURN)
* 🧬 Encoding, decoding, and network optimization

---

### 🛠️ Key Components

| Component             | Purpose                                                             |
| --------------------- | ------------------------------------------------------------------- |
| **getUserMedia()**    | Captures audio/video from the user’s mic/camera                     |
| **RTCPeerConnection** | Establishes and manages the actual P2P media/data connection        |
| **RTCDataChannel**    | Sends arbitrary data (chat messages, files, etc.) over the P2P link |
| **STUN/TURN Servers** | Help peers find and connect through NATs/firewalls                  |

---

### 🔄 How WebRTC Works (Simplified)

1. **Device A and B** want to talk.
2. They use **signaling** (via WebSocket, HTTP, etc.) to exchange:

   * Connection metadata (SDP)
   * Network info (via STUN)
3. Devices attempt a **P2P connection** using `RTCPeerConnection`.
4. If NAT traversal succeeds, they send **audio/video/data directly**.
5. If not, they fall back to **TURN server** (acts as a relay).

---

### 🧱 What WebRTC Handles for You

* NAT traversal
* Jitter buffer
* Codec negotiation (VP8, VP9, H.264)
* Bandwidth adaptation
* Echo cancellation, noise suppression
* Encryption (SRTP, DTLS) ✅ secure by default

---

### 📦 Use Cases

* Video conferencing (Zoom, Google Meet)
* Live streaming platforms
* Online gaming (low-latency voice)
* Remote desktop sharing
* P2P file sharing (e.g. Firefox Send)

---

### ✅ Pros and ❌ Cons

**Pros:**

* Real-time performance
* Peer-to-peer = low latency
* No plugins required
* Built into all major browsers

**Cons:**

* Complex signaling (not part of WebRTC)
* NAT traversal is non-trivial
* Scalability for large group calls requires extra infrastructure (like SFU/MCU)

---

### 🧪 Example Code Snippet (Browser)

```javascript
navigator.mediaDevices.getUserMedia({ video: true, audio: true })
  .then(stream => {
    document.querySelector('video').srcObject = stream;
  });
```

---

### 🧩 Not Included in WebRTC

> WebRTC handles **media & transport**, but **not signaling**.

You must implement signaling separately (e.g., with WebSocket, Firebase, etc.) to exchange metadata between peers.

---

Let me know if you want a **flow diagram**, or how WebRTC compares to alternatives like **Socket.io**, **gRPC**, or **WebSockets**!
