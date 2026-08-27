# EventHub Centralized Configurations (`EventHub-Configurations`)

Centralized configuration repository for the **EventHub - Event & Ticketing Platform**. This repository stores externalized environment configurations served by **Spring Cloud Config Server** to all platform and domain microservices.

- **Repository URI**: `https://github.com/Hansana-Sandamini/EventHub-Configurations.git`
- **Config Server Endpoint**: `http://35.200.169.73:9000` (Production) / `http://localhost:9000` (Local)

---

## 📁 Repository Directory Structure

```
.
├── README.md                           # Documentation for configuration repo
├── application.yaml                    # Shared production config (Eureka cluster, logging, actuators)
├── application-dev.yaml                # Shared local development overrides
├── platform/                           # Platform Infrastructure Configurations
│   ├── api-gateway.yaml                # API Gateway routes, CORS, and port 7000 settings
│   ├── service-registry.yaml           # Eureka Service Registry production configuration
│   └── service-registry-dev.yaml       # Eureka Service Registry local dev configuration
└── services/                           # Domain Microservices Configurations
    ├── user-service.yaml               # User-Service production PostgreSQL datasource & storage
    ├── user-service-dev.yaml           # User-Service local PostgreSQL datasource
    ├── event-service.yaml              # Event-Service production GCP Firestore/MongoDB datasource
    ├── event-service-dev.yaml          # Event-Service local MongoDB datasource
    ├── registration-service.yaml       # Registration-Service production MySQL datasource
    └── registration-service-dev.yaml  # Registration-Service local MySQL datasource
```

---

## ⚙️ Profile Configuration & Resolution

The configurations support two primary execution profiles:

### 1. Production Profile (`default`)
Used during GCP Cloud Run & Compute Engine VM deployment.
- **Eureka Cluster**: Resolves to multi-node internal DNS (`http://vm-node-a.platform:9001/eureka`, `http://vm-node-b.platform:9001/eureka`, `http://vm-node-c.platform:9001/eureka`).
- **Databases**:
  - **User Service**: Cloud SQL / PostgreSQL (`jdbc:postgresql://localhost:5432/db-eventhub`)
  - **Registration Service**: Cloud SQL / MySQL (`jdbc:mysql://localhost:3306/db-eventhub`)
  - **Event Service**: GCP Cloud Firestore / MongoDB TLS cluster via SCRAM-SHA-256.

### 2. Development Profile (`dev`)
Used during local development and testing.
- **Eureka Server**: Resolves to `http://localhost:9001/eureka`.
- **Databases**: Local instances running on custom development ports (`12500` for PostgreSQL, `13500` for MongoDB, `14500` for MySQL).

---

## 🔗 Spring Cloud Config Integration

The **Config Server** loads properties dynamically from this Git repository by defining the search paths for `platform` and `services`:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Hansana-Sandamini/EventHub-Configurations.git
          search-paths: platform,services
```

### Verification Endpoints

Configurations can be inspected via HTTP requests to the Config Server:

```http
GET /user-service/default
GET /event-service/default
GET /registration-service/default
GET /api-gateway/default
GET /service-registry/default
```

---

## 🚀 How to Modify Configurations

1. **Commit Changes**: Push configuration updates directly to the `main` branch of this repository.
2. **Refresh Services**: Downstream Spring Boot microservices can refresh configurations dynamically via Spring Boot Actuator (`POST /actuator/refresh`) or by restarting the process.
