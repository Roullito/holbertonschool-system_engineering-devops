# 3. Scale up

```mermaid
flowchart TD
  U[Users] --> DNSHC[DNS with Health Checks]
  DNSHC --> HAP1[HAProxy 1]
  DNSHC --> HAP2[HAProxy 2]
  HAP1 <---> HAP2

  subgraph Web Tier
    W1[Web 1 Nginx]
    W2[Web 2 Nginx]
  end

  subgraph App Tier
    A1[App 1]
    A2[App 2]
  end

  subgraph Data Tier
    DBP[MySQL Primary]
    DBR[MySQL Replica]
  end

  HAP1 --> W1
  HAP1 --> W2
  HAP2 --> W1
  HAP2 --> W2

  W1 --> A1
  W2 --> A2

  A1 --> DBP
  A2 --> DBP
  DBP --> DBR
```

## Why each addition
- **One new server**: extra capacity for split tiers (web, app, data).
- **Second HAProxy in cluster**: removes LB SPOF; active-active or VIP failover.
- **Split components**: isolate **Web**, **App**, **DB** on their own servers for targeted scaling and security boundaries.


