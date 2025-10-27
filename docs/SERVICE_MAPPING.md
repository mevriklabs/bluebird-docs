# Service & Infrastructure Mapping

This document provides a comprehensive overview of the Mevrik infrastructure, including server locations, service deployments, database connections, and application access points.

---

## 📋 Table of Contents

- [Server & Service Overview](#server--service-overview)
- [Database Cluster](#database-cluster)
- [Application Access](#application-access)
  - [Monitoring & Logging Tools](#-monitoring--logging-tools)
- [Architecture Summary](#architecture-summary)

---

## 🖥️ Server & Service Overview

The following table lists all production servers, their roles, and database dependencies:

| **Server IP**   | **Hostname / Name**  | **Service**                | **Databases Used**                                                                             | **Notes**                                                                                   |
| --------------- | -------------------- | -------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **172.22.7.11** | `dc-worker-gp-api-2` | **Channels API Service**            | MySQL → 172.22.9.30<br>Postgres → 172.22.9.31<br>ClickHouse → 172.22.7.13<br>Redis → 172.22.7.18 | Primary API server handling client requests and real-time operations                        |
| **172.22.7.12** | `dc-worker-gp-api`   | **Channels Worker Service**         | MySQL → 172.22.9.30<br>Postgres → 172.22.9.31<br>Redis → 172.22.7.18 | Background job processor for async tasks, scheduled jobs, and queue workers                 |
| **172.22.7.13** | `dc-worker-gp-api-3` | **Channels Webhook Service**        | MySQL → 172.22.9.30<br>Postgres → 172.22.9.31<br>Redis → 172.22.7.18 | Dedicated webhook receiver for external integrations (Facebook → message,comment)            |
| **172.22.7.18** | `wapp-db-server-1`   | **Redis Service (Docker)** | —                                                                                              | In-memory caching, session,customer,webhook storage, and queue management. Used by all |
| **13.212.9.20** | `ubuntu`             | **Video Call Service**     | —                                                                                              | Standalone video conferencing service accessible at https://video.mevrik.com/dash/dash.php |

---

## 🗄️ Database Cluster

All API, Worker, and Webhook services connect to a shared database cluster:

| **Database Type** | **Server IP**   | **Hostname / Name**      | **Purpose**                                     |
| ----------------- | --------------- | ------------------------ | ----------------------------------------------- |
| **MySQL**         | 172.22.9.30     | `dc-db-chatbot`          | Primary relational database for core application data |
| **PostgreSQL**    | 172.22.9.31     | `dc-db-digital`          | Secondary relational database for specific features    |
| **ClickHouse**    | 172.22.7.13     | `dc-worker-gp-api-3`     | Fast querying for GP KCP/KMP numbers, channels tokens, and static offer details |
| **Redis**         | 172.22.7.18     | `wapp-db-server-1`       | In-memory caching, session storage, and queue management |

---

## 🌐 Application Access

The following applications are available for internal access and testing:

| **Application**                 | **URL / IP**                                     | **Credentials**                               |
| ------------------------------- | ------------------------------------------------ | --------------------------------------------- |
| **GP Inbox (Staging)**          | http://172.22.7.17:3009/inbox                    | Username: `mevrik`<br>Password: `Mevrik@4312` |
| **GP Inbox (Production)**       | https://inbox.mevrik.com/inbox                   | Username: `******`<br>Password: `*****`       |
| **GP Mobile Widget (Staging)**  | https://staging-gp-mobile-widget.mevrik.com/     | —                                             |
| **Chat Widget**                 | https://chat.mevrik.com:4213/                    | —                                             |
| **Video Call Admin Panel**      | https://video.mevrik.com/dash/dash.php           | —                                             |

### 📊 Monitoring & Logging Tools

| **Tool**                        | **URL / IP**                                     | **Purpose**                                   |
| ------------------------------- | ------------------------------------------------ | --------------------------------------------- |
| **Uptime Kuma**                 | https://channels.mevrik.com:4212/dashboard       | Service uptime monitoring and status dashboard |
| **Graylog**                     | http://channels.mevrik.com:9000/search           | Centralized log aggregation and analysis      |

> 💡 **Note**: Credentials for all services and monitoring tools will be provided by the team upon request.

---

## 🧩 Architecture Summary

### Service Distribution

- **API / Worker / Webhook Services** (172.22.7.11–13): Share the same database cluster and form the core application infrastructure
- **Redis Service** (172.22.7.18): Centralized caching and queue management running in Docker, utilized by all services
- **Video Service** (13.212.9.20): Isolated video conferencing service on separate infrastructure

### Network Architecture

```
┌─────────────────────────────────────────────────────────┐
│         Application Layer (172.22.7.11–13)              │
│  ┌─────────────┐  ┌──────────┐  ┌──────────────┐       │
│  │ API Service │  │  Worker  │  │   Webhook    │       │
│  │   (.11)     │  │   (.12)  │  │   Service    │       │
│  │             │  │          │  │    (.13)     │       │
│  └──────┬──────┘  └────┬─────┘  └──────┬───────┘       │
│         │              │               │                │
│         └──────────────┴───────────────┘                │
│                        │                                │
└────────────────────────┼────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    ┌────────┐     ┌──────────┐   ┌────────────┐
    │ MySQL  │     │Postgres  │   │ ClickHouse │
    │ (.30)  │     │  (.31)   │   │   (.13)    │
    └────────┘     └──────────┘   └────────────┘
         
    ┌──────────────────┐       ┌─────────────────┐
    │  Redis (Docker)  │       │  Video Service  │
    │     (.18)        │       │   (13.212.9.20) │
    └──────────────────┘       └─────────────────┘
```

### Key Characteristics

1. **Shared Database Cluster**: All core services (API, Worker, Webhook) use the same multi-database cluster for consistency
2. **Horizontal Scalability**: Services are distributed across multiple servers for load balancing
3. **Separation of Concerns**: Each server has a dedicated role (API, background jobs, webhooks)
4. **Centralized Caching**: Single Redis instance manages all caching and queue operations for API, Worker, and Webhook services
5. **Isolated Video Infrastructure**: Video service runs independently with no Redis or database cluster dependencies

---

**Last Updated**: October 2025  
**Maintained By**: Infrastructure Team