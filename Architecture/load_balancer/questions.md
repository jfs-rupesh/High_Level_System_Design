# Load Balancer Interview Questions & Answers

## 1. What is load balancing, and why is it important?

**Answer**: Load balancing is the process of distributing incoming network traffic across multiple backend servers to ensure efficient utilization, prevent overload, and improve system availability.

- Ensures High Availability: Prevents downtime by redirecting traffic if a server fails.
- Optimizes Resource Utilization: Spreads requests evenly.
- Improves Performance: Routes traffic to the best-performing server.
- Enhances Scalability: Supports horizontal scaling.
- Increases Fault Tolerance: Ensures reliability through redirection.

---

## 2. Explain the difference between Layer 4 and Layer 7 load balancing.

**Answer**:

### Layer 4 (Transport Layer)
- Operates at TCP/UDP level.
- Distributes traffic based on IP and port.
- Faster and simple routing.
- Examples: AWS NLB, HAProxy (L4).

### Layer 7 (Application Layer)
- Operates at HTTP/HTTPS level.
- Routes based on URL paths, headers, cookies.
- Supports SSL termination, caching, auth.
- Examples: AWS ALB, Nginx, Traefik.

**Key Difference**: L4 is faster and less flexible; L7 is intelligent but adds overhead.

---

## 3. How does a load balancer handle high availability and failover?

**Answer**:
- **Health Checks**: Monitors server health (ping, HTTP, TCP).
- **Automatic Failover**: Redirects to healthy servers.
- **Redundancy**: Supports active-active/passive setups.
- **Session Persistence**: Maintains user sessions.
- **Global Load Balancing**: Uses GeoDNS/Anycast.

---

## 4. Compare Round Robin and Least Connections strategies.

**Answer**:

### Round Robin
- Circular distribution of requests.
- Ideal for uniform workloads.
- May overload unevenly sized servers.

### Least Connections
- Chooses server with fewest active connections.
- Suitable for long-lived or varied requests.
- Higher computational overhead.

**Key Difference**: Round Robin is simple but static; Least Connections adjusts dynamically.

---

## 5. What are the advantages of Weighted Load Balancing?

**Answer**:
- Prioritizes servers by capacity.
- High-performance servers get more traffic.
- Custom control over routing.
- Ideal for heterogeneous server environments.
- Examples: Weighted Round Robin, Weighted Least Connections.

---

## 6. When would you use a software load balancer over a hardware one?

**Answer**:

### Software Load Balancer
- Runs on general-purpose hardware.
- Pros: Cost-effective, flexible, container-friendly.
- Cons: Uses system resources, higher latency under load.

### Hardware Load Balancer
- Dedicated appliances.
- Pros: High performance, hardware-accelerated, includes DDoS protection.
- Cons: Expensive, less flexible, harder to scale.

**Use Case**: 
- Software: Cloud-native, scalable services.
- Hardware: Enterprise, high-traffic systems.

---

## 7. How would you design a scalable load balancing solution for a large e-commerce site?

**Answer**:
1. **Multiple Load Balancers**:
   - Use primary and secondary balancers.
   - Distribute traffic with DNS-based global load balancing.

2. **Right Type**:
   - L7 for dynamic content, L4 for DB connections.

3. **Strategies**:
   - Round Robin for static content.
   - Least Connections for dynamic services.

4. **High Availability**:
   - Auto-scaling groups, health checks.

5. **Performance**:
   - Use CDNs, Gzip compression, minification.

---

## 8. What factors should be considered when choosing a load balancing strategy?

**Answer**:
1. **Traffic Pattern**:
   - Round Robin for even traffic.
   - Least Connections for variable complexity.

2. **Server Capacity**:
   - Use Weighted Load Balancing for diverse capacities.

3. **Session Persistence**:
   - Use sticky sessions if needed.

4. **Performance vs Complexity**:
   - L4 for speed.
   - L7 for smart routing.

5. **Scalability**:
   - Cloud-native → cloud LB.
   - On-prem → software/hardware LB.
   

---

## 9. How does a load balancer improve security?

**Answer**:
1. **DDoS Protection**:
   - Blocks malicious spikes.
   - Hardware LBs may include this.

2. **SSL Termination**:
   - Offloads SSL from backend servers.

3. **Access Control**:
   - Firewalls, IP whitelisting.

4. **WAF Integration**:
   - Blocks SQLi, XSS, etc.

5. **Rate Limiting**:
   - Limits requests per client.

---
