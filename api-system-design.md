---
title: "API System Design & Networking / SRE Quick Notes"
category: "System Design"
tags: ["api", "rest", "graphql", "grpc", "networking", "sre", "load-balancing", "interview-prep"]
excerpt: "Interview-ready notes on API design (REST/GraphQL/gRPC), networking layers, load balancing, and SRE reliability patterns."
---

## API Design (5-minute interview section)

### Strategic Framework
- **Time budget**: 5 minutes max on API design; spend more time on core architecture
- **Default to REST** unless specific requirements (GraphQL, gRPC) are mentioned
- **Resource-based** design: Use plural nouns (e.g., `/events`, `/users`), not verbs
- **Keep it concise**: Say "returns Event objects" not every field

---

## API Protocol Choice Matrix

| Protocol | Best For | Trade-offs |
|----------|----------|-----------|
| **REST** | External APIs, user-facing, web/mobile | Default choice for 90% of cases |
| **GraphQL** | Flexible clients, mobile apps, reducing over/under-fetching | More backend complexity, N+1 query problem |
| **gRPC/RPC** | Internal service-to-service, high performance, polyglot | Binary protocol, limited browser support |

---

## REST API Design Patterns

### Resource Modeling
```
GET /events                    # List all
GET /events/{id}               # Get specific
POST /events                   # Create
PUT /events/{id}               # Replace entire resource
PATCH /events/{id}             # Partial update
DELETE /events/{id}            # Delete
```

### HTTP Methods (Idempotency matters!)
- **GET** – Idempotent, safe, no side effects
- **POST** – NOT idempotent, creates new resources
- **PUT** – Idempotent, replace entire resource
- **PATCH** – Not guaranteed idempotent
- **DELETE** – Idempotent

### Where to Pass Data
- **Path params** (`/events/123`): Required resource identifier
- **Query params** (`?city=NYC&limit=10`): Optional filters, sorting, pagination
- **Request body**: Complex payloads for POST/PUT/PATCH

### Example: Booking POST request
```
POST /events/123/bookings?notify=true
{
  "tickets": [{"section": "VIP", "quantity": 2}],
  "payment_method": "credit_card"
}
```
- Event ID in path (required)
- Notify flag in query (optional behavior)
- Booking details in body (core data)

---

## HTTP Status Codes (Interview Cheat Sheet)

### Success (2xx)
- **200** – OK, general success
- **201** – Created, new resource made

### Client Error (4xx)
- **400** – Bad Request
- **401** – Unauthorized (auth required)
- **403** – Forbidden (authenticated but not allowed)
- **404** – Not Found
- **429** – Too Many Requests (rate limited)

### Server Error (5xx)
- **500** – Internal Server Error
- **502** – Bad Gateway (upstream issue)
- **503** – Service Unavailable(Server overloaded, maintenance, degradation )
- **504** – Gateway Timeout (Request exceeded timeout threshold)

**Interview tip**: Bucket by 2xx/4xx/5xx rather than memorizing each code.

---

## Advanced API Patterns

### Pagination
**Offset-based** (simple, good for small datasets)
```
GET /events?offset=20&limit=10
```

**Cursor-based** (stable for real-time data, no duplicates on concurrent adds)
```
GET /events?cursor=cmd9atj3p000007ky19w1dpy2&limit=10
```

### Versioning
- **URL versioning** (preferred for interviews): `/v1/events`, `/v2/events`
- **Header versioning**: `Accept-Version: v2` (cleaner URLs, harder to test)

### Authentication & Authorization
- **API Keys**: For server-to-server, 3rd party dev access (not user sessions)
- **JWT Tokens**: For user-facing APIs; stateless, carries user context
- **RBAC**: Assign roles (customer, venue_manager, admin) with specific permissions
- **Security rule**: Never trust user_id from request body; derive from auth token

### Rate Limiting
- Per-user: 1000 req/hour
- Per-IP: 100 req/hour (unauthenticated)
- Endpoint-specific: 10 booking attempts/min
- Return **429 Too Many Requests** when exceeded

---

## GraphQL (When to Use)

### Problem it Solves
- **Over-fetching**: Client gets more data than needed
- **Under-fetching**: Client needs multiple requests
- **Fast iteration**: Frontend can request new fields without backend changes

### Example Query
```graphql
query {
  event(id: "123") {
    name
    date
    venue { name, address }
    tickets { section, price, available }
  }
}
```

### Gotcha: N+1 Problem
Querying 100 events with venues = 1 query for events + 100 queries for venues = **101 queries**. Solution: Implement dataloader batching.

---

## gRPC / RPC (Internal APIs)

### Why it's Fast
- **Binary serialization** (Protocol Buffers) vs JSON → 10x throughput
- **HTTP/2** multiplexing
- **Type-safe** generated code (compile-time error detection)

### When to Use
- Internal service-to-service communication
- High throughput, latency-critical paths
- Polyglot environments (multiple languages)
- Streaming support (bidirectional)

### Typical Setup
- **Public APIs**: REST
- **Internal APIs**: gRPC

---

---

# Networking & SRE Reliability Patterns

## Networking Layers (OSI Model)

| Layer | Examples | Key Concept |
|-------|----------|------------|
| **Layer 3 (Network)** | IP | Routing, addressing |
| **Layer 4 (Transport)** | TCP, UDP, QUIC | Reliability, ordering, flow control |
| **Layer 7 (Application)** | HTTP, DNS, WebSocket | User-facing protocols |

---

## Transport Layer: TCP vs UDP

### TCP (Transmission Control Protocol)
✅ Reliable delivery, ordered, error-checked  
❌ Higher latency, connection overhead  
**Use for**: Everything where data integrity matters (default choice)

### UDP (User Datagram Protocol)
✅ Fast, low latency, connectionless  
❌ No delivery guarantee, no ordering  
**Use for**: Real-time video/audio, VoIP, DNS, gaming, live streams (where occasional loss is OK)

---

## Application Layer Protocols

### HTTP/HTTPS
- **Stateless** by design (good for horizontal scaling)
- **Request/response** model
- Built on TCP
- HTTPS adds TLS encryption (security in transit)
- Security note: Don't trust request body without validation

### WebSockets
- **Persistent, bidirectional** TCP connection
- Initiated via HTTP upgrade
- Ideal for: Real-time apps, games, live chat, multiplayer
- Caveat: Infrastructure (load balancers, firewalls) must support WebSocket upgrade
- **Load balancer**: Use **Layer 4** (not Layer 7) to maintain persistent connection to same backend

### Server-Sent Events (SSE)
- Server pushes messages to client over single HTTP connection
- Simpler than WebSocket (unidirectional server→client)
- Auto-reconnect with message ID tracking
- Good for: Live notifications, live auction updates

### WebRTC
- **Peer-to-peer** communication between browsers
- Uses **STUN** (NAT traversal) + **TURN** (relay fallback)
- Over **UDP**
- Use only for: Audio/video calling, conferencing (too complex otherwise)

---

## Load Balancing

### Two Approaches

#### Client-Side Load Balancing
- Client picks which server to call
- Uses service registry (e.g., Redis Cluster knows all nodes)
- Fast (no extra hop), but only works with small number of controlled clients
- Examples: Redis Cluster, DNS, gRPC client-side LB

#### Dedicated Load Balancer
- Separate server routes requests to backends
- Fast server list updates, works with uncontrolled clients
- Adds one network hop per request

### Layer 4 vs Layer 7 Load Balancers

| Feature | L4 (TCP/UDP) | L7 (HTTP) |
|---------|-------------|----------|
| **Connection** | Persistent TCP to same backend | Terminates & creates new ones |
| **Routing** | IP/port only | URL, headers, cookies |
| **Speed** | Very fast, minimal inspection | More CPU-intensive |
| **Persistence** | Built-in (same connection = same server) | Requires sticky sessions |
| **Best for** | WebSocket, high performance | HTTP, flexible routing |

**Interview tip**: WebSockets → L4 LB. HTTP (REST, API) → L7 LB.

### Load Balancing Algorithms
- **Round Robin**: Sequential across servers (default for stateless)
- **Random**: Randomly pick server
- **Least Connections**: Pick server with fewest active connections (good for SSE/WebSocket)
- **Least Response Time**: Fastest server wins
- **IP Hash**: Same client IP → same server (session persistence)

---

## Handling Network Failures (SRE Patterns)

### 1. Timeouts & Retries
```
if request_timeout_exceeded:
  retry_with_backoff()
```
- Set realistic timeout (don't wait forever)
- Retry transient failures
- **Must have**: Exponential backoff + jitter (randomness)
  - Without jitter: all clients retry together = thundering herd = worse
  - With jitter: staggered retries = system recovers

### 2. Idempotency (Critical for Retries!)
❌ Bad: Retry payment → double charge
✅ Good: Idempotency key = user_id + date → charge only once

**Pattern**: Include idempotency key in request, check on server before processing

### 3. Circuit Breakers
Prevent cascading failures when calling external services/databases

**States**:
- **Closed** (normal): Requests pass through
- **Open** (failing): Requests fail immediately (fail fast, reduce load)
- **Half-Open** (testing): Send one test request to check recovery

**When to apply**:
- External API calls
- Database connections
- Service-to-service calls
- Timeout-prone operations

**Benefits**:
- Fail fast (don't wait for timeout)
- Reduce load on struggling service
- Self-healing (auto-test recovery)
- Prevent thundering herd

---

## Regionalization & Latency

### The Physics Problem
- Light through fiber ≈ 2/3 speed of light = ~200k km/s
- NYC → London (5,600 km) = **56ms minimum latency** (just from physics!)

### Solutions

#### Content Delivery Network (CDN)
- Edge locations near users
- Cache static/cacheable content
- Used for: Images, videos, search results (fast, frequent reads)

#### Regional Partitioning
- Partition data by geographic region
- Each region has its own database
- Example: Uber drivers/riders in Miami served by Miami region
- Ultra-fast queries (local DB)

---

## Common Failure Modes & Mitigations

| Failure Mode | Cause | Mitigation |
|--------------|-------|-----------|
| **Transient slowness** | Temporary overload | Retry with backoff |
| **Cascading failure** | One service down → rest fail | Circuit breaker |
| **Thundering herd** | All clients retry at same time | Add jitter to backoff |
| **Database can't start** | Firehose of retries pins it down | Circuit breaker + gradual drain |
| **Data loss** | Network partition | Replication across regions |

---

## Interview Checklist

### API Design Section
- [ ] Protocol choice justified (REST default)
- [ ] Resources mapped to core entities
- [ ] Key endpoints sketched
- [ ] Auth/authn approach mentioned
- [ ] Rate limiting called out
- [ ] Keep it **under 5 minutes**

### Networking/Reliability Deep Dives
- [ ] Mention timeouts & exponential backoff
- [ ] Idempotency for critical operations (payments, etc.)
- [ ] Circuit breakers for external calls
- [ ] Load balancer choice (L4 for WebSocket, L7 for HTTP)
- [ ] Health checks & failover
- [ ] Regional partitioning for geo-distribution

---

## Key Takeaways

1. **Default to REST** for external APIs; GraphQL/gRPC only if needed
2. **Idempotent by default**: Design APIs and operations to be retryable
3. **Fail gracefully**: Timeouts, retries with backoff, circuit breakers
4. **L4 LB for stateful** (WebSocket), **L7 LB for HTTP**
5. **Proximity matters**: Use CDN, regional partitioning to reduce latency
6. **Prevent cascades**: Circuit breakers save systems at 3am
