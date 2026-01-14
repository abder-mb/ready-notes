# Management App - High-Level Design (HLD)

## Executive Summary

The Management App is a centralized control plane for a Data Mesh ecosystem, enabling domain teams to manage data assets, projects, permissions, and cross-account sharing. The architecture leverages AWS services (DataZone, Glue, Lake Formation) combined with a microservices-based management application running on EKS.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  ON-PREMISES                                                     │
│  • Development Teams (GitLab, Cognito)                          │
│  • Authorized Users & Administrators                            │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ AWS Direct Connect (1-10 Gbps)
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  AWS REGION (us-east-1)                                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  LAYER 1: USER INTERACTION (Outside VPC)                  │ │
│  │                                                             │ │
│  │  Client ──► Route 53 (DNS) ──► API Gateway                │ │
│  │                                   │                         │ │
│  │                                   ├─ AWS WAF               │ │
│  │                                   ├─ Lambda Authorizer      │ │
│  │                                   └─ VPC Link              │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  LAYER 2: APPLICATION (Inside VPC)                        │ │
│  │                                                             │ │
│  │  EKS Cluster (12 Microservices)                            │ │
│  │  ├─ Domain Service          ├─ Data Quality Service       │ │
│  │  ├─ Project Service         ├─ Lineage Tracking Service   │ │
│  │  ├─ Asset Service           ├─ Iceberg Table Service      │ │
│  │  ├─ Catalog Service         └─ Other Services             │ │
│  │  ├─ Permission Service                                     │ │
│  │  ├─ Subscription Service                                   │ │
│  │  ├─ Lake Formation Service                                 │ │
│  │  └─ Cross-Account Share Service                            │ │
│  │                                                             │ │
│  │  VPC Endpoints (Private Access)                            │ │
│  │  ├─ EventBridge   ├─ Step Functions  ├─ S3  ├─ DynamoDB  │ │
│  │  ├─ Glue          ├─ DataZone        ├─ KMS └─ STS        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  LAYER 3: ORCHESTRATION (Outside VPC)                     │ │
│  │                                                             │ │
│  │  EventBridge (Event Bus)                                   │ │
│  │  └─ Receives events from EKS pods                          │ │
│  │     ├─ ProjectCreated   ├─ AssetPublished                 │ │
│  │     ├─ SubscriptionRequest  └─ PermissionChanged          │ │
│  │                                                             │ │
│  │  Step Functions (Workflow Orchestration)                   │ │
│  │  └─ Triggered by EventBridge rules                         │ │
│  │     ├─ Project Promotion Workflow                          │ │
│  │     ├─ Cross-Account Share Workflow                        │ │
│  │     └─ Asset Publication Workflow                          │ │
│  │                                                             │ │
│  │  Lambda (Scheduled Functions)                              │ │
│  │  ├─ Hourly: Data Quality Checks                            │ │
│  │  ├─ Daily: PROD→DEV Data Sync                              │ │
│  │  └─ Weekly: Cleanup Old Snapshots                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  LAYER 4: AWS SERVICE INTEGRATION                         │ │
│  │                                                             │ │
│  │  ├─ Amazon DataZone (SMUS)                                 │ │
│  │  ├─ AWS Glue (Data Catalog + ETL)                          │ │
│  │  ├─ AWS Lake Formation (Permissions + Access Control)      │ │
│  │  ├─ AWS Athena (Query Engine - Iceberg)                    │ │
│  │  ├─ Amazon Redshift (Data Warehouse)                       │ │
│  │  ├─ AWS RAM (Cross-Account Sharing)                        │ │
│  │  ├─ AWS CloudFormation (IaC Deployments)                   │ │
│  │  ├─ AWS IAM (Identity & Access Management)                 │ │
│  │  ├─ AWS KMS (Key Management)                               │ │
│  │  └─ AWS Secrets Manager (Credentials)                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  LAYER 5: DATA PERSISTENCE                                │ │
│  │                                                             │ │
│  │  Application State (DynamoDB)                              │ │
│  │  ├─ DeploymentHistory  ├─ PromotionRequests               │ │
│  │  ├─ ApprovalWorkflows  └─ ConfigurationCache              │ │
│  │                                                             │ │
│  │  Data Lake (S3)                                            │ │
│  │  ├─ Raw Data          ├─ Processed Data                    │ │
│  │  ├─ Metadata          └─ Iceberg Snapshots                 │ │
│  │                                                             │ │
│  │  Managed Services                                          │ │
│  │  ├─ Glue Data Catalog (Table Metadata)                     │ │
│  │  └─ DataZone (Domain Metadata)                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Components

### Layer 1: User Interaction
- **Route 53**: DNS resolution for API endpoint
- **API Gateway**: Regional endpoint with rate limiting
- **AWS WAF**: Request filtering and DDoS protection
- **Lambda Authorizer**: JWT/OAuth token validation, IAM policy generation
- **VPC Link**: Secure bridge to internal ALB

### Layer 2: Application Services
12 microservices running in EKS, handling:
- Domain and project management
- Asset discovery and publishing
- Permission and subscription workflows
- Lake Formation integration
- Cross-account sharing
- Data quality monitoring
- Lineage tracking
- Iceberg table management

**Infrastructure:**
- EKS Cluster (multi-AZ, auto-scaling)
- Internal Application Load Balancer
- VPC Endpoints for private AWS service access
- IAM Roles for Service Accounts (IRSA) for authentication

### Layer 3: Orchestration
- **EventBridge**: Central event bus for publishing/subscribing to domain events
- **Step Functions**: Long-running workflows with approval gates, retries, error handling
- **Lambda**: Scheduled automation (hourly, daily, weekly tasks)

### Layer 4: AWS Service Integration
Direct boto3 SDK integration with:
- DataZone, Glue, Lake Formation for data governance
- CloudFormation for infrastructure deployment
- IAM/KMS/Secrets Manager for security

### Layer 5: Persistence
- **DynamoDB**: Application state, audit trails, deployment history
- **S3**: Data lake, Iceberg snapshots, configuration
- **Glue Catalog**: Table metadata and schema
- **DataZone**: Domain and asset metadata

---

## Data Flow

### Flow 1: EKS → EventBridge
1. EKS pod publishes event (ProjectCreated, AssetPublished, etc.)
2. Event sent via VPC Endpoint to EventBridge
3. EventBridge matches event pattern against rules
4. Triggering Step Functions or other targets

### Flow 2: Step Functions → EKS
1. Step Functions state machine receives trigger
2. Makes HTTP POST call to EKS service via VPC Link
3. Request routed through ALB to target pod
4. Pod executes action and returns response

### Flow 3: EKS/Step Functions → AWS Services
1. Code in EKS pods or Lambda functions
2. Uses boto3 SDK to call AWS service APIs (DataZone, Glue, etc.)
3. Authenticated via IRSA (EKS) or execution role (Lambda)
4. Service processes request and persists to Layer 5

---

## Security

### Network Security
- Direct Connect for on-prem connectivity (no internet)
- VPC with private subnets only (no public IPs)
- VPC Endpoints for AWS service access (no internet gateway)
- VPC Link for API Gateway integration (no public ALB)
- AWS WAF on API Gateway (rate limiting, SQL injection protection)

### Access Control
- Lambda Authorizer for API authentication/authorization
- IAM roles and policies for service-to-service communication
- Lake Formation for fine-grained data access (column/row level)
- RBAC in Kubernetes for pod access control

### Encryption
- AWS KMS for encryption at rest (S3, DynamoDB, Glue Catalog)
- TLS/SSL for encryption in transit (all API calls)
- Secrets Manager for managing sensitive data

### Audit & Monitoring
- CloudWatch Logs for all services
- EventBridge rules for audit events
- DynamoDB streams for state change tracking
- S3 access logging

---

## High Availability & Disaster Recovery

### Availability
- Multi-AZ EKS cluster with worker nodes in 2+ availability zones
- Horizontal Pod Autoscaling (HPA) for dynamic scaling
- Application Load Balancer distributes traffic across pods
- DynamoDB global tables for replication (optional for multi-region)

### Disaster Recovery
- Regular snapshots of DynamoDB tables
- S3 cross-region replication for data lake
- CloudFormation templates for infrastructure recovery
- AWS Backup for automated backup management

---

## Cost Optimization

- **EKS**: Fargate for serverless container execution (optional)
- **DynamoDB**: On-demand billing (pay-per-request)
- **Lambda**: Scheduled functions only run on demand
- **Step Functions**: Pay per state transition
- **VPC Endpoints**: Per-endpoint hourly charge (amortized across services)

---

## Deployment & Operations

### Infrastructure as Code
- Terraform or CloudFormation for AWS resources
- Helm charts for Kubernetes deployments
- GitOps for deployment pipeline (GitLab → ArgoCD → EKS)

### Monitoring & Alerting
- CloudWatch dashboards for metrics
- SNS topics for critical alerts
- X-Ray for distributed tracing
- Custom metrics from applications

### Logging
- CloudWatch Logs for centralized logging
- Log Insights for querying and analysis
- S3 for long-term log retention

---

## Scalability

- **EKS Cluster**: Auto-scales from 3 to 100+ nodes based on demand
- **Pods**: HPA scales each service independently (replicas 1-50)
- **DynamoDB**: On-demand billing scales automatically
- **Lambda**: Concurrent execution limit (reserved concurrency for critical functions)
- **EventBridge**: Handles 1000s of events/second
- **Step Functions**: Handles 1000s of concurrent executions

---

## Next Steps

1. **Phase 1**: Deploy core infrastructure (VPC, EKS, IAM roles)
2. **Phase 2**: Deploy management app microservices
3. **Phase 3**: Integrate with DataZone and Glue
4. **Phase 4**: Implement EventBridge and Step Functions workflows
5. **Phase 5**: Setup monitoring, logging, and alerting
6. **Phase 6**: Multi-region deployment (optional)

