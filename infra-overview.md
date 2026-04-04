# Bulqit Infrastructure Overview

## Production Architecture

```mermaid
graph TD
    Users["Homeowners & Service Providers"]
    Users --> WEB

    subgraph platform["Bulqit Platform"]
        direction LR
        WEB["<u>Web App</u><br/>Vercel · Next.js"]
        DB[("<u>Database</u><br/>Neon · PostgreSQL")]
        CMS["<u>Content</u><br/>Sanity CMS"]
        WEB --> DB
        WEB --> CMS

        subgraph workers["Background Services · Fly.io"]
            TW["Workflows"]
            ZS["CRM Sync"]
            NW["Notifications"]
        end
        DB --> ZS & NW
    end

    subgraph services["Third-Party Services"]
        direction LR
        AP["<u>Auth & Payments</u><br/>Clerk · Stripe"]
        CO["<u>Communications</u><br/>Twilio · SendGrid"]
        AN["<u>Analytics & Monitoring</u><br/>Segment · BigQuery · Sentry<br/>FullStory · Google Analytics"]
        ORCH["<u>Workflow Engine</u><br/>Temporal Cloud"]
        CRM["<u>CRM & Growth</u><br/>Zoho · Referral Factory"]
    end

    WEB --> AP
    WEB -.-> AN
    TW --> ORCH
    ZS --> CRM
    NW --> CO
```

## Service Inventory

```mermaid
graph LR
    subgraph core1["Platform · Hosting"]
        direction TB
        V["Vercel · Web + File Storage"]
        N["Neon · Database"]
        S["Sanity · CMS"]
    end

    subgraph core2["Platform · Services"]
        direction TB
        F["Fly.io · Background Services"]
        TC["Temporal Cloud · Workflows"]
        GM["Google Maps · Location"]
    end

    subgraph auth["Auth & Payments"]
        direction TB
        CL["Clerk · Authentication"]
        ST["Stripe · Payments"]
    end

    subgraph comms["Communications"]
        direction TB
        TW["Twilio · SMS · Chat · Support"]
        SG["SendGrid · Email"]
    end

    subgraph crm["CRM & Growth"]
        direction TB
        ZO["Zoho CRM"]
        RF["Referral Factory · Referral Program"]
        ZB["ZeroBounce · Email Validation"]
    end

    subgraph analytics["Analytics & Monitoring"]
        direction TB
        SE["Sentry · Error Tracking"]
        FS["FullStory · Session Replay"]
        SG2["Segment · Event Tracking"]
        BQ["BigQuery · Data Warehouse"]
        GA["Google Analytics"]
    end


```

**Service inventory notes:**

- **BigQuery** is a Segment destination, not a direct code integration. Whether data flows there depends on Segment dashboard configuration.

## Deployment Model

```mermaid
graph LR
    subgraph vendor["Vendor · FutureWorks"]
        UP["Upstream Repo<br/>Private"]
    end

    subgraph bulqit["Bulqit-Controlled"]
        MIRROR["bul-website<br/>GitHub Mirror"]
        INFRA["bul-infra<br/>Disaster Recovery"]
    end

    subgraph hosting["Hosting"]
        VC["Vercel<br/>Web App"]
        FLY["Fly.io<br/>Workers"]
        NEON["Neon<br/>Database"]
    end

    UP -- "GitHub Actions<br/>hourly mirror" --> MIRROR
    MIRROR -- "auto-deploy<br/>on push" --> VC
    MIRROR -- "manual deploy" --> FLY
    INFRA -- "provisioning" --> VC
    INFRA -- "automated" --> FLY
    INFRA -- "automated" --> NEON
```

## Reconstruction Flow

```mermaid
graph TD
    subgraph onetime["One-Time Setup"]
        O0["Configure credentials<br/>Install CLI tools"]
        O1["<u>mise setup</u><br/>Authenticate all CLIs"]
        O2["<u>mise init</u><br/>Initialize Terraform<br/>Bootstrap env files"]
        O0 --> O1 --> O2
    end

    START(["mise reconstruct"])

    subgraph preflight["Pre-flight Checks"]
        PF1["Env files exist?"]
        PF2["Production guards OK?"]
    end

    subgraph auto["Interactive · 11 Steps"]
        S1["1 · Clone mirror repo"]
        S2["2 · Provision Vercel project + domain"]
        S3["3 · Link workspace to Vercel"]
        S4["4 · Install Neon integration"]
        S5["5 · Push Vercel env vars"]
        S6["6 · Deploy web app"]
        S7["7 · Create Fly.io org"]
        S8["8 · Provision Fly workers"]
        S9["9 · Push Fly secrets"]
        S10["10 · Deploy workers"]
        S11["11 · Health checks"]
    end

    subgraph manual["Manual Steps"]
        M1["Restore database backup"]
        M2["Import Sanity CMS dataset"]
        M3["Configure Clerk,<br/>Temporal, SendGrid"]
        M4["Re-enable GitHub Actions"]
        M5["Redeploy with updated config"]
        M6["DNS cutover"]
    end

    O2 --> START
    START --> PF1 --> PF2
    PF2 --> S1 --> S2 --> S3 --> S4 --> S5 --> S6
    S6 --> S7 --> S8 --> S9 --> S10 --> S11
    S11 --> M1 --> M2 --> M3 --> M4 --> M5 --> M6

    style onetime fill:#e3f2fd,stroke:#1976d2
    style preflight fill:#f0f0f0,stroke:#999
    style auto fill:#e8f5e9,stroke:#4caf50
    style manual fill:#fff3e0,stroke:#ff9800
```
