# API Gateway – Interview Questions & Detailed Answers

---

### 1. What is an API Gateway, and why is it used?

An API Gateway is a server that acts as an intermediary between clients and backend services. It manages API requests, handles routing, applies middleware like authentication and rate limiting, and transforms responses. It simplifies client interactions, enforces security policies, and supports system scalability.

---

### 2. How does an API Gateway differ from a Load Balancer?

A Load Balancer distributes incoming network traffic across multiple servers to ensure high availability and reliability. It works at Layer 4 or 7. An API Gateway, operating at Layer 7, provides additional features like request routing, authentication, caching, and response transformation.

---

### 3. What are the key benefits of using an API Gateway?

* **Security**: Centralizes authentication and authorization.
* **Rate Limiting & Throttling**: Protects backend services.
* **Load Balancing**: Manages traffic efficiently.
* **Caching**: Reduces response time and load.
* **Monitoring & Logging**: Tracks usage and performance.
* **Protocol Translation**: Converts between protocols like HTTP and gRPC.

---

### 4. How does an API Gateway handle authentication and authorization?

API Gateways enforce security through methods like:

* **API Keys**
* **OAuth 2.0 & JWT**
* **Mutual TLS (mTLS)**
* **LDAP & SAML**
  The gateway checks credentials before forwarding the request. Unauthorized requests are blocked.

---

### 5. What middleware functionality can an API Gateway handle?

* JWT/OAuth2 Authentication
* Rate limiting & throttling
* IP allow/deny lists
* SSL termination
* CORS management
* Request logging
* API versioning

---

### 6. How does request routing work inside an API Gateway?

Routing is typically based on:

* URL path (e.g., /users/\*)
* HTTP method
* Query parameters
* Request headers
  The gateway uses these to direct traffic to the correct backend service.

---

### 7. Can an API Gateway handle protocol transformation (e.g., REST to gRPC)?

Yes. Many gateways (e.g., Google Cloud Endpoints, Kong) can translate REST calls into internal gRPC calls or other formats, enabling clients to use simpler protocols while microservices optimize internally.

---

### 8. What is the typical lifecycle of a request through an API Gateway?

1. Request validation
2. Middleware (auth, rate limiting, logging)
3. Routing to appropriate backend service
4. Protocol transformation (if needed)
5. Response transformation
6. Optional response caching

---

### 9. Explain rate limiting and throttling in API Gateways.

* **Rate Limiting**: Restricts API calls per client over time (e.g., 100 requests/minute).
* **Throttling**: Allows excess requests but slows them down.
* **Burst Limits**: Allow short-term spikes within limits.
  Common algorithms include Token Bucket, Leaky Bucket, and Sliding Window.

---

### 10. What caching strategies can be implemented in an API Gateway?

* **In-memory caching** (e.g., Redis)
* **Response caching** (e.g., for GET requests)
* **Edge caching/CDN**
* **Per-route caching**
* **TTL-based expiration**
  Caching reduces load and improves latency.

---

### 11. How does an API Gateway improve security against DDoS attacks?

* Rate limiting and throttling
* IP whitelisting/blacklisting
* WAF integration
* CAPTCHA & bot detection
* TLS termination
  These techniques prevent resource exhaustion and ensure backend availability.

---

### 12. How does the API Gateway contribute to enforcing CORS policies?

It sets HTTP headers like `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers` in the response to control cross-origin requests.

---

### 13. What are the common algorithms used for rate limiting?

* **Token Bucket**: Tokens refill periodically; requests consume tokens.
* **Leaky Bucket**: Fixed-rate processing, buffers excess.
* **Fixed Window**: Limits per time bucket (e.g., per minute).
* **Sliding Window**: More precise rate tracking over time.

---

### 14. How does TLS termination work in an API Gateway setup?

The gateway decrypts incoming HTTPS traffic and forwards plain HTTP to backend services. This offloads TLS handling from internal services and centralizes security policies.

---

### 15. When should you use an API Gateway in a microservices architecture?

When you need:

* A unified entry point
* Centralized authentication
* Rate limiting & caching
* Logging & monitoring
* Simplified client interactions

---

### 16. When is an API Gateway unnecessary or overkill?

* Monolithic apps with one backend
* Minimal middleware or routing needs
* Early-stage MVPs where simplicity is key

---

### 17. How would you design an API Gateway for a large-scale system with millions of users?

* Deploy distributed gateway instances
* Use auto-scaling (e.g., with Kubernetes)
* Enable caching & rate limiting
* Multi-region deployment with failover
* Central logging and metrics

---

### 18. How does an API Gateway integrate with service discovery?

Gateways can dynamically resolve service endpoints using tools like Consul, Eureka, or Kubernetes service discovery. This enables real-time routing updates.

---

### 19. What challenges might arise when implementing an API Gateway?

* **Single point of failure** → Use multiple instances
* **Latency** → Optimize config, enable caching
* **Complexity** → Use management platforms (e.g., Apigee)
* **Scalability** → Use stateless instances & horizontal scaling
* **Versioning** → Adopt URI-based or header-based versioning

---

### 20. What are the pros and cons of centralized vs decentralized (microgateway) models?

* **Centralized**: Easier control and consistency; risk of bottleneck.
* **Decentralized**: Scalability and autonomy; harder to govern.

---

### 21. How do you handle versioning of APIs behind an API Gateway?

* URI versioning: `/v1/users`
* Header-based: `Accept: application/vnd.myapi.v1+json`
* Query parameters: `/users?version=1`

---

### 22. How do you monitor and log API Gateway traffic effectively?

* Enable structured logging (e.g., JSON)
* Integrate with APM tools (Datadog, Prometheus)
* Use tracing (OpenTelemetry, Jaeger)
* Collect metrics on latency, errors, and throughput

---

### 23. How do you scale an API Gateway horizontally?

API Gateways are stateless and can be scaled horizontally by running multiple instances behind a load balancer. Auto-scaling rules can be set based on traffic.

---

### 24. How would you deploy an API Gateway for global traffic with low latency?

* Use regional gateway deployments
* Route via GeoDNS or CDN
* Sync configs across regions
* Deploy closest to the user

---

### 25. How would you implement JWT-based authentication in an API Gateway?

* Parse and validate the JWT token in the gateway
* Check signature, expiration, and claims
* Forward user ID or scopes to the backend via headers

---

### 26. What is mutual TLS (mTLS), and how does an API Gateway support it?

mTLS ensures both client and server authenticate each other via TLS certificates. Gateways handle the handshake and certificate validation before routing.


---

### 27. How would you handle multi-tenancy with an API Gateway?

* Use subdomains or headers to identify tenants
* Apply tenant-specific policies (auth, rate limits)
* Isolate routing per tenant

---

### 28. How does the API Gateway interact with a service mesh like Istio or Linkerd?

The gateway can serve as the mesh ingress controller, handling external traffic and forwarding it into the mesh. Internal traffic is then managed by the mesh.

---

### 29. Can an API Gateway enforce business logic? Should it?

It *can*, but it *shouldn’t*. Business logic belongs in services. Gateways should focus on edge concerns like routing, security, and observability.

---

### 30. What are some popular open-source and managed API Gateways?

* **Open Source**: Kong, Tyk, Express Gateway, KrakenD
* **Managed**: AWS API Gateway, Azure API Management, Apigee (GCP)
  Each has trade-offs between control, cost, and ecosystem integration.

---
