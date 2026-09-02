# Azure Cloud Engineering Foundations

This future track prepares a .NET-focused full-stack candidate to design, deploy, secure, operate, and discuss a modern React and ASP.NET Core application on Azure.

## Scope

The track focuses on the Azure responsibilities a full-stack engineer is likely to own: selecting managed services, securing application access, deploying repeatably, operating reliably, and defending technical trade-offs.

It is not a catalogue of Azure products or a certification study guide. Kubernetes and platform-landing-zone design are awareness topics unless a target role explicitly requires deeper platform engineering expertise.

## Learning Outcomes

By the end of this track, you should be able to:

- Select appropriate Azure hosting, data, and messaging services for a .NET and React application.
- Use Microsoft Entra ID, managed identities, RBAC, configuration, and Key Vault to minimise secret-based access.
- Explain how an application is deployed, configured, scaled, monitored, and recovered.
- Diagnose common production failures with Application Insights, Azure Monitor, logs, metrics, and traces.
- Make defensible trade-offs around reliability, security, performance, and cost.

## Modules

### Module 1 - Azure Core, Identity, and Governance

**Priority:** Critical  
**Prerequisites:** .NET Platform and Development Model awareness

- Subscriptions, management groups, resource groups, regions, and availability zones
- Azure Resource Manager, resource providers, and subscription quotas
- Microsoft Entra ID: users, app registrations, service principals, and managed identities
- RBAC, least privilege, resource locks, tags, and naming conventions
- Azure Policy awareness and common governance constraints
- Azure Portal, Azure CLI, and infrastructure-as-code awareness; provision a small baseline with Bicep or Terraform

### Module 2 - Hosting .NET and React Applications

**Priority:** Critical  
**Prerequisites:** Module 1

- App Service, Azure Functions, Container Apps, AKS, and Static Web Apps
- Selecting App Service, Functions, Container Apps, AKS, or Static Web Apps for a workload
- Docker and container fundamentals for .NET applications
- Deployment slots, revisions, health checks, scaling, and configuration
- Hosting React separately versus serving it through ASP.NET Core
- Static frontend hosting, CDN awareness, and SPA routing fallback

### Module 3 - Configuration, Secrets, and Application Identity

**Priority:** Critical  
**Prerequisites:** Modules 1 and 2

- Configuration versus secrets
- Azure Key Vault and Key Vault references
- Managed identity from ASP.NET Core to Azure resources
- OAuth/OIDC awareness: app registrations, delegated versus application permissions, scopes, app roles, and SPA/API authorization boundaries
- Local development with `DefaultAzureCredential`
- Environment variables and App Service or Container Apps configuration
- Secret rotation, least privilege, and preventing secrets from reaching frontend bundles

### Module 4 - Data, Storage, and Caching

**Priority:** High  
**Prerequisites:** Modules 2 and 3

- Azure SQL Database: sizing, network access, backups, and connection resiliency
- Azure Storage: Blob, Queue, Table, and File services; choosing the right one
- Azure Cache for Redis: cache-aside, expiration, invalidation, and failure modes
- Cosmos DB awareness and when not to choose it
- Connection pooling, parameterized queries, migrations, and data protection

### Module 5 - Messaging and Background Processing

**Priority:** High  
**Prerequisites:** Modules 2 and 3

- Service Bus queues, topics, subscriptions, dead-letter queues, and sessions
- Event Grid versus Service Bus versus Storage Queues
- Azure Functions and worker services for background work
- At-least-once delivery, duplicates, idempotency, retries, and poison messages
- Outbox-pattern awareness

### Module 6 - Networking and Application Security

**Priority:** High  
**Prerequisites:** Modules 1-4

- Public endpoints, VNets, subnets, NSGs, private endpoints, and private DNS
- Azure Front Door and Application Gateway awareness
- CORS across React and API hosts
- TLS, custom domains, WAF awareness, and DDoS-protection awareness
- Network access for Azure SQL, Storage, Key Vault, and Container Apps
- Authentication architecture: Entra, cookies/tokens, and SPA redirect URIs

### Module 7 - Observability, Reliability, and Incident Response

**Priority:** Critical  
**Prerequisites:** Modules 2-5

- Application Insights, Azure Monitor, Log Analytics, and distributed tracing
- Structured logging, correlation IDs, metrics, alerts, and dashboards
- Availability tests, health checks, dependency telemetry, and release annotations
- Autoscaling, availability zones, retries, timeouts, and circuit-breaker awareness
- Diagnosing latency, dependency failures, and failed deployments
- Backup, restore, recovery expectations, and RTO/RPO awareness

### Module 8 - Delivery, IaC, Cost, and Architecture Trade-offs

**Priority:** High  
**Prerequisites:** Modules 1-7

- CI/CD with GitHub Actions or Azure DevOps
- Bicep or Terraform, parameters, environments, and infrastructure drift
- Managed identities for deployments and workload access
- Build, test, security scanning, deployment, smoke tests, and rollback
- Cost drivers: compute plans, databases, logs, bandwidth, storage, and idle resources
- Azure Well-Architected Framework: reliability, security, cost optimisation, operational excellence, and performance efficiency

## Suggested Learning Sequence

1. Start with Modules 1-3 to understand Azure resource organisation, application identity, secrets, hosting decisions, and repeatable provisioning.
2. Study Modules 4 and 5 to design data and asynchronous workflows.
3. Add Module 6 when scenarios include private networking, enterprise security, or ingress design.
4. Use Modules 7 and 8 to learn how the application is operated and delivered.

## Practical Deliverables

- Defend a hosting choice for a React SPA and ASP.NET Core API, comparing App Service, Functions, Container Apps, and Static Web Apps.
- Explain how a .NET API accesses Azure SQL, Storage, Key Vault, and Service Bus without embedded credentials.
- Design a reliable message-processing workflow that handles duplicate delivery and dead-letter messages.
- Diagnose an API latency or dependency failure using traces, logs, metrics, and alerts.
- Explain an infrastructure-as-code and CI/CD flow, including validation, deployment, smoke testing, and rollback.
- Identify the primary cost drivers in a small production architecture and propose proportionate reductions.

## Scope Boundaries

- Detailed C#, SQL, API design, frontend, security theory, and testing remain in the existing .NET and frontend tracks; this track covers Azure implementation and operational consequences.
- AKS is service-selection awareness rather than core material. Container Apps is the default container-focused path for typical full-stack scenarios.
- Advanced network architecture, platform landing zones, and organisation-wide governance belong in a later platform-engineering or architecture track.
