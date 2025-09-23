# What happens when you type `https://www.google.com` and press Enter

*By <Jules MOLEINS>*

Recruiters love this question because it reveals how well you understand the full web stack. Below is a concise, end‑to‑end walkthrough that touches the layers you are likely to be asked about: **DNS**, **TCP/IP**, **firewalls**, **HTTPS/TLS**, **load balancers**, **web servers**, **application servers**, and **databases**. I close with a compact diagram you can keep in your head during interviews.

---

## 1) DNS: turning a name into an address

Your browser first needs the IP address for `www.google.com`.

1. **Cache checks**: Browser cache → OS cache → local DNS resolver (often your ISP or a public resolver).
2. **Iterative resolution** (if cache misses): the resolver asks a chain of servers:
   - **Root name servers** (where to find `.com`),
   - **.com TLD name servers** (where to find `google.com`),
   - **Authoritative name servers for `google.com`**, which return one or more **A/AAAA records** (IPv4/IPv6) or a **CNAME** that maps `www.google.com` to another hostname.
3. **TTL (time to live)** controls how long results are cached to reduce latency and load.

**Result:** your machine learns an IP (actually, usually several) for `www.google.com`.

---

## 2) TCP/IP: establishing a connection to the right port

Armed with an IP, your system opens a **TCP** connection to the **server IP on port 443**, the default for **HTTPS**. TCP performs the **three‑way handshake** (SYN → SYN‑ACK → ACK) which sets up a reliable, ordered, byte stream over **IP**. Packet routing and NAT happen along the way, but they’re transparent to your app.

---

## 3) Firewall: only the right traffic gets in

Between the public Internet and Google’s edge, traffic passes through **network firewalls** and **security groups** that allow inbound connections **only** to permitted ports (here: **TCP/443**). Packets not matching the policy are dropped. This protects internal services and reduces the attack surface.

---

## 4) HTTPS / TLS: encrypting and authenticating the session

Once TCP is up, the browser starts a **TLS handshake** with the server (still over port 443):

- The server presents an **X.509 certificate** proving ownership of `www.google.com` (signed by a trusted Certificate Authority).
- Client and server agree on a **cipher suite** and establish **shared keys**.
- From then on, application data (HTTP requests and responses) is **encrypted** and has **integrity protection**.

**Why it matters:** confidentiality (no eavesdropping), integrity (no tampering), and server authenticity (no easy impersonation).

---

## 5) Load balancer: distributing the request

At Google scale, your HTTPS connection typically terminates on an **edge load balancer** (or a fleet). The LB may:

- **Distribute** traffic to a healthy backend using algorithms like round‑robin or least‑connections.
- **Offload TLS** (decrypt at the edge), optionally re‑encrypt to backends.
- Enforce policies (rate limiting, header normalization) and health checks.

The LB forwards your request to a **web server** in an appropriate region/cluster.

---

## 6) Web server: HTTP optimization and reverse proxying

A **web server** (e.g., Google’s own stack or software like Nginx) terminates HTTP, serves **static assets** efficiently (HTML, CSS, JS, images), and **reverse‑proxies** dynamic requests to the **application server**. It may also apply compression, caching, and connection pooling to upstreams.

---

## 7) Application server: business logic

The **application server** runs the code that decides how to fulfill your request (search, Gmail, Maps…). It reads cookies/headers, checks auth, runs business logic, calls internal services, and may render HTML or emit JSON for a client app.

---

## 8) Database: persistent state

To answer your request, the application server queries persistent stores: **databases** (SQL/NoSQL), **caches** (e.g., in‑memory key‑value), or **search indexes**. Databases are typically **replicated** for durability and performance (primary for writes, replicas for reads). The app merges the results and returns a response to the web server, which sends it back through the LB to your browser.

---

## 9) The round trip in one breath

- **DNS** resolves `www.google.com` → server IP.
- **TCP** connects to **port 443**.
- **Firewalls** allow only permitted traffic.
- **TLS** secures the channel (HTTPS).
- **Load balancer** picks a healthy backend.
- **Web server** serves static content and proxies dynamic requests.
- **Application server** generates the response.
- **Database** (and caches) supply persistent data.
- Response flows back to your browser, which renders the page (parsing, DOM, CSSOM, JS).

---

## Diagram (GitHub‑safe Mermaid)

The diagram below captures the required elements: DNS resolution, TCP 443, encryption, firewall, load balancing, web server, app server, and database.

```mermaid
flowchart TD
  Browser[Browser] -->|Resolve www.google.com| DNSR[DNS Resolver]
  DNSR -->|A or AAAA record| Browser
  Browser -->|TCP 443 HTTPS| FW[Firewall allowing 443]
  FW --> LB[Load Balancer]
  LB --> WS[Web Server]
  WS --> APP[Application Server]
  APP --> DB[Database]
  note right of Browser: Encrypted via TLS
```

---

### If you want me to go deeper…
For front‑end roles we can expand on parsing, preloading, TLS resumption, H2/H3 multiplexing, or caching headers. For SRE roles we can dive into anycast, BGP, edge POPs, health checks, failover, and observability (SLIs/SLOs).
