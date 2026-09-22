# Module 04: Networking — End-to-End Web Request Lifecycle

---

## 1. The Global Lifecycle Architecture

```
[ Browser UI ]
      │ (1. Parse URL & HSTS)
[ DNS Resolution Pipeline ]
      │ (2. Browser Cache -> OS -> Resolver -> Root -> TLD -> Auth)
[ ARP Protocol ]
      │ (3. Resolve Gateway IP to Gateway MAC Address)
[ TCP 3-Way Handshake ]
      │ (4. SYN -> SYN-ACK -> ACK over Port 443)
[ TLS 1.3 Cryptographic Handshake ]
      │ (5. Key Share & Certificate Validation)
[ HTTP/2 or HTTP/3 Encrypted Request ]
      │ (6. GET / HTTP/2)
[ Edge Infrastructure & CDN / Load Balancer ]
      │ (7. Anycast Routing -> L4/L7 Reverse Proxy)
[ Application Server & Database Execution ]
      │ (8. Web Runtime -> SQL / Cache Query)
[ HTTP Response Stream ]
      │ (9. 200 OK + Gzip/Brotli HTML Stream)
[ Critical Rendering Path (Browser) ]
      ▼ (10. DOM + CSSOM -> Render Tree -> Layout -> Paint)
```

---

## 2. Step-by-Step Execution Mechanics

### Step 1: Input Processing & HSTS Evaluation
1. **URL Parsing**: The browser parses the input string into constituent parts:
   - Protocol: `https` (Default port: `443`)
   - Hostname: `www.example.com`
   - Path: `/`
2. **HSTS (HTTP Strict Transport Security)**: The browser checks its internal preloaded HSTS list. If present, it forces HTTPS immediately before issuing any network requests, preventing SSL-stripping man-in-the-middle attacks.

### Step 2: Domain Name Resolution (DNS)
To locate the server, the browser resolves `www.example.com` through a four-tier cache hierarchy:
1. **Browser DNS Cache**: Checked first (stores DNS records for short TTLs).
2. **OS DNS Cache & Hosts File**: Checked if browser cache misses (e.g. `getaddrinfo()` system call).
3. **Recursive DNS Resolver (ISP / Anycast)**: Queries Root Servers (`.`), TLD Servers (`.com`), and finally Authoritative Nameservers to return the `A`/`AAAA` record (`93.184.216.34`).

### Step 3: Layer 2 Link Resolution (ARP)
If the destination IP resides outside the local subnet, the host must forward the packet to the local **Default Gateway Router**.
- The host checks its local **ARP Cache** for the gateway's MAC address.
- If absent, it broadcasts an **ARP Request** (`Who has 192.168.1.1?`) and caches the unicast reply.

### Step 4: TCP Connection Establishment
The client initiates a **3-Way Handshake** with `93.184.216.34:443`:
1. Client sends `SYN (Seq=X)`.
2. Server responds with `SYN-ACK (Seq=Y, Ack=X+1)`.
3. Client acknowledges with `ACK (Seq=X+1, Ack=Y+1)`.
The socket state transitions to `ESTABLISHED`.

### Step 5: TLS 1.3 Cryptographic Handshake
1. Client sends `ClientHello` advertising supported cipher suites and public key parameters (ECDHE).
2. Server returns `ServerHello` with its public key parameter, digital certificate, and cryptographic signature.
3. Client verifies the certificate chain against local Root Certificate Authorities.
4. Both sides compute the shared symmetric encryption key. Handshake latency = **1 Round Trip Time (1-RTT)**.

### Step 6: HTTP Request Dispatch & Infrastructure Routing
1. The browser formats the HTTP request (e.g., `GET / HTTP/2`), encrypts it with the symmetric key, and transmits it.
2. **Anycast BGP**: Directs packet to the nearest geographic Point of Presence (PoP).
3. **CDN / Reverse Proxy**: Terminates TLS and evaluates caching headers.
4. **L7 Load Balancer**: Forwards request to an active backend service instance via consistent hashing or round-robin.

### Step 7: Server Execution & Response Generation
1. Web server processes HTTP headers, validates session cookies or JWT authorization tokens.
2. Database query executed against primary or read replica via a connection pool.
3. Response payload compressed using gzip or Brotli and streamed back: `HTTP/2 200 OK`.

### Step 8: Critical Rendering Path (Browser Pipeline)
Once the browser receives the raw byte stream:
1. **DOM Tree Construction**: Characters $\to$ Tokens $\to$ Nodes $\to$ Document Object Model (DOM).
2. **CSSOM Tree Construction**: External stylesheets and `<style>` blocks parsed into CSS Object Model.
3. **Render Tree Creation**: Combines DOM and CSSOM, omitting non-visible elements (`display: none`).
4. **Layout (Reflow)**: Calculates the exact geometry, position, and dimensions of each node on screen.
5. **Painting**: Converts vector boxes into actual pixels on the GPU/screen buffer.
6. **Compositing**: Draws layered surfaces in correct stacking order (`z-index`).
