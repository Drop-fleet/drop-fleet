# 2. System architecture

> **UNVERIFIED HYPOTHESIS.** No service code, manifests, or configs were
> readable. This diagram is put together **only from the repo and component
> names in the audit brief** so the team has something concrete to correct.
> Every box and arrow needs confirming. Dashed edges mark things the brief
> mentions only as examples ("e.g. Telnyx").

```mermaid
flowchart LR
  subgraph Clients
    DA[Driver apps<br/>iOS / Android]
    CA[Consumer apps<br/>iOS / Android]
    PWA[PWAs<br/>dispatcher / consumer?]
    WEB[Marketing site]
  end

  subgraph "Drop Fleet (argocd-df)"
    FS[drop-fleet-thedropapp-server]
  end
  subgraph "Drop Delivery (argocd-dd)"
    DS[drop-technologies-thedropapp-server]
  end
  subgraph Shared?
    API2[atyantik-drop-api.v2]
    TRK[atyantik-drop-tracking]
    POS[pos-integration]
    CHAT[Socket.IO chat]
    ALERT[alert]
  end

  DB[(Database(s)?)]
  Q[[Queue / cache?]]
  MON[argocd-monitoring]

  DA --> FS & DS
  DA -- location pings --> TRK
  CA --> DS
  PWA --> FS & DS
  DA & CA <--> CHAT
  FS & DS --> DB
  API2 --> DB
  TRK --> DB
  POS --> DS
  FS & DS -.-> Q

  FS & DS -.-> MAPS[(Maps / geocoding API)]
  FS & DS -.-> SMS[(SMS, e.g. Telnyx)]
  FS & DS -.-> PAY[(Payments provider)]
  FS & DS -.-> PUSH[(Push: APNs / FCM)]
  POS -.-> POSV[(POS vendor APIs)]
  MON --> ALERT
```

## Questions the code needs to answer

- Is `atyantik-drop-api.v2` replacing the `*-thedropapp-server` backends, or
  running alongside them? Which one do the mobile apps call in production?
- One database per product, or one shared database?
- Is there a queue (Redis/BullMQ, SQS, RabbitMQ), or is all async work done by
  cron/polling?
- Does POS sync use vendor webhooks or polling? (This matters a lot for cost.)
- Which maps/geocoding provider is used, and are results cached?
