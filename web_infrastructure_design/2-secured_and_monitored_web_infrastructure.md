# 2. Secured and monitored web infrastructure

```mermaid
flowchart TD
  U[Users] --> DNSR[DNS Resolver]
  U --> HAP[HAProxy HTTPS Termination]

  FW_LB[Firewall at Load Balancer]
  FW_S1[Firewall at Server1]
  FW_S2[Firewall at Server2]

  subgraph Server1
    W1[Web 1 Nginx] --> A1[App 1]
    A1 --> DBP[MySQL Primary]
    MON1[Monitoring Agent]
  end

  subgraph Server2
    W2[Web 2 Nginx] --> A2[App 2]
    A2 --> DBR[MySQL Replica]
    MON2[Monitoring Agent]
  end

  MONLB[Monitoring Agent LB]

  HAP --> W1
  HAP --> W2
  DBP --> DBR

  U -. allowed ports .-> FW_LB
  W1 -. allowed ports .-> FW_S1
  W2 -. allowed ports .-> FW_S2

  MON1 --> MONB[Monitoring Backend]
  MON2 --> MONB
  MONLB --> MONB
```

## Why each addition
- **3 firewalls**: enforce least-privilege networking (only required ports).
- **1 SSL certificate**: serve **HTTPS** for privacy, integrity, and server auth.
- **3 monitoring clients**: ship metrics/logs/traces to a backend (Sumologic/ELK/Prometheus).

## What firewalls are for
- Control inbound/outbound access: e.g. **443** to LB from Internet; **80/8080** app only inside; **3306** DB only from app; **22** only from admin IP or bastion.

## Why HTTPS
- Prevents eavesdropping/tampering and authenticates the site to users.

## What monitoring is used for
- Track **availability, latency, errors, resource usage**, and **alert** on issues.

## How data is collected
- Agents **tail logs** and **scrape metrics** (exporters or status endpoints) and send to the monitoring backend.

## Monitor web server QPS
- Enable **Nginx stub_status** or parse access logs; scrape counters at intervals; compute **QPS** and alert on thresholds.

## Issues to explain
- **SSL at LB only**: backend traffic is plain HTTP; use **end-to-end TLS** if required.
- **Single writable MySQL**: primary remains a **write SPOF**; prepare automated failover or multi-primary.
- **Same components everywhere**: increases blast radius; **separate roles** improves isolation.

