# Module 04: Networking — API Architectures: REST, GraphQL, gRPC & WebSockets

---

## 1. RESTful Architectural Constraints

Representational State Transfer (REST) is an architectural style governed by six foundational constraints:
1. **Client-Server Separation**: Decouples user interface concerns from data storage concerns.
2. **Statelessness**: Every incoming request must contain all contextual information (including authentication) required to process it. The server stores no session state between calls.
3. **Cacheability**: Responses must explicitly designate themselves as cacheable or non-cacheable to prevent clients from retrieving stale data.
4. **Uniform Interface**: Standardized URI resource paths (`/users/42`), standard HTTP methods, and representation schemas.
5. **Layered System**: Clients cannot tell whether they are connected directly to the end application server or to an intermediary proxy, load balancer, or CDN.

---

## 2. HTTP Methods, Safety & Idempotency

- **Safe Method**: Does not mutate server state (read-only).
- **Idempotent Method**: Executing the request $N$ times consecutively produces the exact same server resource state as executing it once ($f(f(x)) = f(x)$).

```
┌──────────┬──────────┬──────────────┬─────────────────────────────────────────────────┐
│ Method   │ Safe?    │ Idempotent?  │ Intended Semantic                               │
├──────────┼──────────┼──────────────┼─────────────────────────────────────────────────┤
│ GET      │ YES      │ YES          │ Retrieve resource representation without side   │
│          │          │              │ effects.                                        │
│ POST     │ NO       │ NO           │ Create a new subordinate resource or execute an │
│          │          │              │ arbitrary action.                               │
│ PUT      │ NO       │ YES          │ Completely replace an existing resource or      │
│          │          │              │ create it at an explicit URI.                   │
│ PATCH    │ NO       │ NO           │ Apply partial delta modifications to a resource.│
│ DELETE   │ NO       │ YES          │ Remove a resource. Repeated calls continue to   │
│          │          │              │ leave the resource deleted.                     │
└──────────┴──────────┴──────────────┴─────────────────────────────────────────────────┘
```

---

## 3. HTTP Status Codes Reference

```
1xx: Informational │ 2xx: Success       │ 3xx: Redirection   │ 4xx: Client Error  │ 5xx: Server Error
───────────────────┼────────────────────┼────────────────────┼────────────────────┼───────────────────
101 Switching Prots│ 200 OK             │ 301 Moved Perm     │ 400 Bad Request    │ 500 Internal Error
                   │ 201 Created        │ 302 Found (Temp)   │ 401 Unauthorized   │ 502 Bad Gateway
                   │ 204 No Content     │ 304 Not Modified   │ 403 Forbidden      │ 503 Service Unavail
                   │                    │                    │ 404 Not Found      │ 504 Gateway Timeout
                   │                    │                    │ 409 Conflict       │
                   │                    │                    │ 429 Rate Limited   │
```

### Critical Distinctions:
- **`401 Unauthorized` vs. `403 Forbidden`**:
  - `401`: Missing or invalid authentication credentials ("Who are you? Log in first").
  - `403`: Identity is authenticated, but lacks permissions for this resource ("I know who you are, but you cannot access this").
- **`502 Bad Gateway` vs. `504 Gateway Timeout`**:
  - `502`: An edge proxy/gateway received an invalid or crashed response from the upstream application server.
  - `504`: The upstream application server failed to respond within the gateway's timeout window.

---

## 4. API Architectural Paradigms Comparison

```
REST (JSON over HTTP)                   GRAPHQL (Client Query)                  gRPC (Protobuf / HTTP/2)
┌───────────────────────────────┐       ┌───────────────────────────────┐       ┌───────────────────────────────┐
│ • Resource-oriented URIs      │       │ • Single endpoint (/graphql)  │       │ • Protocol Buffers binary serialization│
│ • Multiple network roundtrips │       │ • Client requests exact fields│       │ • High performance RPC        │
│ • Over/Under-fetching risk    │       │ • Zero over/under-fetching    │       │ • Strict schema contract (.proto)│
└───────────────────────────────┘       └───────────────────────────────┘       └───────────────────────────────┘
```

### Architectural Feature Matrix
| Dimension | REST | GraphQL | gRPC | WebSockets |
| :--- | :--- | :--- | :--- | :--- |
| **Protocol** | HTTP/1.1 or HTTP/2 | Typically HTTP/1.1 | **HTTP/2 exclusively** | TCP (Upgraded from HTTP) |
| **Data Format** | JSON / XML (Text) | JSON (Text) | **Protocol Buffers (Binary)** | Text or Binary frames |
| **Communication** | Request-Response | Request-Response | Unary, Client/Server Streaming, Bi-directional | **Full-Duplex Persistent** |
| **Performance** | Medium (Text serialization overhead) | Medium | **Extremely High** (Compact binary serialization) | High (Zero header overhead per message) |
| **Best Used For** | Public public APIs, CRUD web services | Mobile clients, dashboards aggregating multiple services | **Microservices internal communication**, low-latency RPC | Real-time chat, trading feeds, collaborative editing |
