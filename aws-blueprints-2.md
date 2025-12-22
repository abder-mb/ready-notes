


# Architecture Review: Gen AI Blueprints for SageMaker Unified Studio

## Strengths (Pros)

**Well-Structured Layering**
Your architecture demonstrates clear separation of concerns with distinct layers for data ingestion, model operations, inference patterns, and cross-cutting services. This modular approach enhances maintainability and enables independent scaling of components.

**Comprehensive Inference Patterns**
The inclusion of multiple inference types (serverless, real-time, asynchronous, and batch) shows thoughtful consideration for different workload characteristics and cost optimization strategies.

**Strong DevOps Integration**
The Model Handler blueprint with ECS, Step Functions, and CodeBuild indicates a mature approach to ML operations with automated deployment pipelines and orchestration capabilities.

**Security-First Design**
The Inference-Base blueprint with API Gateway, Lambda Orchestration, and Lambda Authorizer demonstrates proper authentication and authorization patterns.

**Observability**
Including Check and Debug services (SageMaker Clarify and Ground Truth) shows commitment to model monitoring and data quality validation.

## Weaknesses (Cons)

**Missing Data Governance**
- No Lake Formation or AWS Glue Data Catalog for data discovery and access control
- Absence of data lineage tracking mechanisms
- No clear data versioning strategy

**Limited Experiment Tracking**
- SageMaker Experiments or MLflow integration not visible
- No clear model versioning or registry workflow beyond SageMaker Model Registry

**Incomplete Monitoring Stack**
- Missing CloudWatch integration for metrics and logs
- No X-Ray for distributed tracing
- Limited visibility into model performance degradation

**Network Architecture Gaps**
- VPC boundaries and security groups not defined
- No PrivateLink endpoints shown for AWS services
- Direct Connect path unclear despite on-premises component

**Cost Management Absence**
- No AWS Cost Explorer or tagging strategy
- Missing SageMaker Savings Plans consideration
- No autoscaling policies defined

## Architecture Improvements

### 1. Enhanced Data Layer

**Add Data Governance Components:**
- Integrate AWS Lake Formation for fine-grained access control
- Add AWS Glue Data Catalog for metadata management
- Implement Amazon DataZone for data discovery and collaboration
- Include AWS Glue DataBrew for data quality rules

**Data Versioning:**
- Add DVC (Data Version Control) or use S3 versioning with manifests
- Implement dataset snapshots with timestamps in S3 prefixes

### 2. Strengthen MLOps Pipeline

**Experiment Tracking:**
- Add SageMaker Experiments to the Model Handler blueprint
- Integrate MLflow for experiment comparison and model registry
- Include SageMaker Feature Store for feature management and reusability

**CI/CD Enhancements:**
- Add automated model testing stages (unit, integration, shadow deployment)
- Include SageMaker Pipelines for end-to-end ML workflow orchestration
- Add approval gates before production deployment

### 3. Comprehensive Monitoring & Observability

**Add Monitoring Stack:**
- CloudWatch Dashboards for unified metrics view
- CloudWatch Logs Insights for log analysis
- X-Ray for distributed tracing across services
- SageMaker Model Monitor for data drift and model quality

**Alerting:**
- SNS topics for critical alerts (already have EventBridge/SNS)
- PagerDuty or similar integration for on-call rotation
- Automated rollback triggers based on performance thresholds

### 4. Network & Security Architecture

**Network Design:**
- Define VPC architecture with public/private subnets
- Add VPC endpoints (PrivateLink) for S3, SageMaker, API Gateway
- Implement AWS Transit Gateway for on-premises connectivity
- Use AWS Network Firewall for egress filtering

**Security Enhancements:**
- Add AWS WAF in front of API Gateway
- Implement AWS Secrets Manager for credentials (if not already implicit)
- Add AWS KMS key management diagram for encryption at rest
- Include AWS IAM Identity Center for user federation
- Add Security Hub for compliance monitoring

### 5. Cost Optimization

**Add Cost Management:**
- Implement tagging strategy (project, environment, cost-center)
- Add AWS Cost Explorer integration
- Use SageMaker Savings Plans for predictable workloads
- Implement S3 Intelligent-Tiering for data storage
- Add autoscaling policies for inference endpoints

### 6. Additional Components

**Model Registry Enhancement:**
- Add model approval workflow
- Implement model cards for documentation
- Include A/B testing framework in inference layer

**Disaster Recovery:**
- Add cross-region replication for critical models
- Implement backup strategy for feature store and model artifacts
- Define RTO/RPO requirements

**Data Pipeline:**
- Consider adding Amazon Kinesis for streaming data ingestion
- Add AWS Glue ETL jobs for data transformation
- Include Amazon EMR for large-scale data processing if needed

### 7. Blueprint Refinements

**Inference-Base:**
- Add Amazon CloudFront for API caching and DDoS protection
- Include AWS AppSync for GraphQL APIs if needed
- Add Amazon ElastiCache for response caching

**Cross-Blueprints:**
- Add shared VPC/networking blueprint
- Create IAM roles and policies blueprint
- Add cost allocation and tagging blueprint

## Recommended Architecture Additions

```
┌─────────────────────────────────────────────┐
│ Governance & Security Layer                 │
├─────────────────────────────────────────────┤
│ Lake Formation | KMS | Secrets Manager      │
│ Security Hub | GuardDuty | IAM Identity     │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ Observability Layer                         │
├─────────────────────────────────────────────┤
│ CloudWatch | X-Ray | SageMaker Monitor      │
│ Cost Explorer | Trusted Advisor             │
└─────────────────────────────────────────────┘
```

Your architecture provides a solid foundation. These improvements would elevate it to production-grade enterprise standards with enhanced security, governance, cost optimization, and operational excellence.




--------------

# Refined Architecture Review: Gen AI Blueprints for SageMaker Unified Studio

Given your context (consumer linked account, Gen AI focus, SMUS catalog integration), here's a refined analysis:

## Updated Strengths

**SMUS Integration Ready**
Your architecture is well-positioned for SageMaker Unified Studio consumption patterns with proper separation between data ingestion and model operations.

**Gen AI Inference Patterns**
Multiple inference types support various Gen AI use cases - real-time for chatbots, async for document processing, batch for large-scale generation tasks.

**Foundation Model Ready**
SageMaker JumpStart integration in the Data Layer shows readiness for foundation model deployment.

## Critical Gaps for Gen AI

### 1. **Missing Amazon Bedrock Integration**
For Gen AI workloads, Bedrock should be a primary component:
- Bedrock Knowledge Bases for RAG (Retrieval Augmented Generation)
- Bedrock Agents for autonomous task execution
- Bedrock Guardrails for safety and content filtering
- Foundation Model access (Claude, Titan, etc.)

### 2. **Vector Database Absence**
Essential for Gen AI/RAG patterns:
- Amazon OpenSearch Serverless (vector engine)
- Amazon Aurora PostgreSQL with pgvector
- Integration with embeddings generation

### 3. **Prompt Management**
No prompt engineering or management layer visible:
- Prompt templates storage (S3/DynamoDB)
- Prompt versioning
- Prompt optimization workflow

### 4. **LLM Evaluation Framework**
Missing model evaluation specific to Gen AI:
- SageMaker Clarify for LLM bias detection
- Custom evaluation metrics (ROUGE, BLEU, human feedback)
- A/B testing framework for prompt variations

## Enhanced Architecture with Missing Components

### Updated Data Layer Blueprint

```
┌────────────────────────────────────────────────────────┐
│ Data Layer + Gen AI Extensions                         │
├────────────────────────────────────────────────────────┤
│ Existing: Athena, Glue ETL, SageMaker JumpStart       │
│ Existing: SageMaker Data Wrangler                      │
│                                                         │
│ ADD: AWS Glue Data Catalog (SMUS shared catalogs)     │
│ ADD: Amazon Bedrock Knowledge Bases                    │
│ ADD: OpenSearch Serverless (vector store)              │
│ ADD: Document processing (Textract integration)        │
└────────────────────────────────────────────────────────┘
```

### New Blueprint: Gen AI RAG Pipeline

```
┌────────────────────────────────────────────────────────┐
│ RAG Pipeline Blueprint                                  │
├────────────────────────────────────────────────────────┤
│                                                         │
│  S3 (Documents) → Textract/Lambda → Embeddings        │
│                        ↓                                │
│                 OpenSearch Serverless                   │
│                        ↓                                │
│             Bedrock Knowledge Base                      │
│                        ↓                                │
│                 API Gateway → Lambda                    │
│                        ↓                                │
│              Bedrock (Foundation Models)                │
│                   + Guardrails                          │
└────────────────────────────────────────────────────────┘
```

### Enhanced Model Handler Blueprint

```
┌────────────────────────────────────────────────────────┐
│ Model Handler + Gen AI Extensions                      │
├────────────────────────────────────────────────────────┤
│ Existing: SageMaker AI, ECS, Step Functions           │
│                                                         │
│ ADD: Bedrock Model Evaluation                          │
│ ADD: Prompt Template Store (DynamoDB/S3)               │
│ ADD: SageMaker Experiments (LLM metrics)               │
│ ADD: Human Feedback Loop (A2I)                         │
│ ADD: SageMaker Feature Store (prompt features)         │
└────────────────────────────────────────────────────────┘
```

### Enhanced Inference Layer

```
┌────────────────────────────────────────────────────────┐
│ Inference - Base + Gen AI                              │
├────────────────────────────────────────────────────────┤
│ Existing: API Gateway → Lambda → Bedrock              │
│          → Lambda Authorizer                           │
│                                                         │
│ ADD: Bedrock Guardrails (content filtering)           │
│ ADD: DynamoDB (conversation history/context)           │
│ ADD: ElastiCache (prompt/response caching)             │
│ ADD: Bedrock Agents (for autonomous workflows)         │
└────────────────────────────────────────────────────────┘
```

## Critical Lineages & Integrations

### Data Lineage Flow

```
SMUS Catalog (shared data)
    ↓
AWS Glue Data Catalog (consumer account)
    ↓
Athena (query) / Glue ETL (transform)
    ↓
S3 Storage (refined data)
    ↓
Document Processing → Embeddings Generation
    ↓
OpenSearch Serverless (vector store)
    ↓
Bedrock Knowledge Base (RAG)
    ↓
Inference Endpoints
```

### Model Lineage Flow

```
SageMaker JumpStart (Foundation Models)
    ↓
SageMaker Training (fine-tuning) OR Bedrock (direct use)
    ↓
SageMaker Experiments (tracking)
    ↓
Model Evaluation (Clarify + custom metrics)
    ↓
SageMaker Model Registry
    ↓
Approval Workflow (Step Functions)
    ↓
SageMaker Endpoint OR Bedrock Inference
    ↓
Model Monitor (drift detection)
```

### Prompt Lineage Flow

```
Prompt Template (S3/DynamoDB)
    ↓
Prompt Versioning (Git/S3 versioning)
    ↓
Prompt Optimization (A/B testing)
    ↓
Evaluation Metrics (human feedback)
    ↓
Production Prompt (deployed)
    ↓
Monitoring (usage analytics)
```

## Enhanced Cross-Blueprints Services

### Add Gen AI Specific Services

```
┌────────────────────────────────────────────────────────┐
│ Cross-Blueprints Services + Gen AI                     │
├────────────────────────────────────────────────────────┤
│ Existing: EventBridge, SNS, CloudWatch, KMS           │
│                                                         │
│ ADD: AWS Secrets Manager (API keys, credentials)      │
│ ADD: Amazon Augmented AI (A2I) - human review         │
│ ADD: AWS Step Functions (orchestration workflows)     │
│ ADD: Amazon DynamoDB (conversation state/history)      │
│ ADD: AWS CloudTrail (audit logging)                    │
│ ADD: Amazon CloudFront (API caching/DDoS)              │
└────────────────────────────────────────────────────────┘
```

## Key Integration Points

### 1. SMUS Catalog Integration

```
Producer Account (SMUS Central)
    ↓ (share via Lake Formation or Resource Access Manager)
Consumer Account (Your Studio)
    ↓
AWS Glue Data Catalog (local catalog with shared references)
    ↓
Your blueprints consume shared data
```

**If you need to share back to SMUS:**
- Add Lake Formation in your account
- Create Resource Access Manager shares
- Tag data assets appropriately

### 2. Bedrock Integration Points

**Knowledge Base Integration:**
```
S3 Documents → Bedrock Knowledge Base → OpenSearch Serverless
                        ↓
                API Gateway → Lambda → Bedrock Runtime
```

**Agent Integration:**
```
Bedrock Agent → Lambda Functions (tools)
              → API Gateway (external APIs)
              → SageMaker Endpoints
```

### 3. Monitoring & Observability Integration

```
All Services → CloudWatch Logs
            → CloudWatch Metrics
            → X-Ray (distributed tracing)
            ↓
CloudWatch Dashboards (unified view)
            ↓
EventBridge Rules → SNS → Email/Slack
```

### 4. Security Integration

```
API Gateway → WAF (DDoS protection)
           → Lambda Authorizer
           → Secrets Manager (credentials)
           → KMS (encryption)
           ↓
CloudTrail (audit logs)
```

## Recommended Additional Blueprints

### Blueprint: Document Processing Pipeline
```
S3 (docs) → Lambda → Textract → Lambda (processing)
                                    ↓
                         Embeddings (Bedrock/SageMaker)
                                    ↓
                         OpenSearch Serverless
```

### Blueprint: Conversation Management
```
API Gateway → Lambda → DynamoDB (session state)
                    → ElastiCache (context caching)
                    → Bedrock (LLM inference)
```

### Blueprint: Evaluation & Feedback
```
Inference Results → A2I (human review)
                  → DynamoDB (feedback storage)
                  → Step Functions (evaluation workflow)
                  → CloudWatch (metrics)
```

## Security & Governance Additions

### VPC Architecture (High-Level)
```
┌──────────────────────────────────────┐
│ VPC                                   │
├──────────────────────────────────────┤
│ Private Subnets:                     │
│  - SageMaker Notebooks/Studio        │
│  - Lambda functions                   │
│  - OpenSearch Serverless             │
│                                       │
│ VPC Endpoints:                        │
│  - S3 Gateway Endpoint               │
│  - Bedrock Interface Endpoint        │
│  - SageMaker Interface Endpoint      │
│  - Secrets Manager Endpoint          │
└──────────────────────────────────────┘
```

### IAM Structure (High-Level)
```
Service Roles:
- SageMaker Execution Role (Studio, training, endpoints)
- Lambda Execution Role (inference, processing)
- Glue Service Role (ETL jobs)
- Step Functions Execution Role

Data Access:
- S3 bucket policies (least privilege)
- Glue Data Catalog resource policies
- Bedrock model access policies
```

## Revised Architecture Diagram Additions

**Add these boxes to your existing diagram:**

1. **Data Layer**: 
   - AWS Glue Data Catalog (with SMUS integration note)
   - Amazon Bedrock Knowledge Base
   - OpenSearch Serverless

2. **New Blueprint - RAG Pipeline**:
   - Document Processing flow
   - Embeddings generation
   - Vector store integration

3. **Model Handler**:
   - SageMaker Experiments
   - Prompt Management (DynamoDB)
   - Human Feedback (A2I)

4. **Inference Layer**:
   - Bedrock Guardrails
   - DynamoDB (conversation state)
   - ElastiCache (caching)

5. **Cross-Blueprints**:
   - Secrets Manager
   - CloudTrail
   - CloudFront

This refined architecture is now Gen AI-focused, SMUS-ready, and includes proper lineage tracking while respecting your constraints as a consumer linked account.





-------






-----------------

# Updated Gen AI Architecture for SageMaker Unified Studio

## Alternative Vector Store Solution

Since OpenSearch Serverless is not allowed, use **Amazon Aurora PostgreSQL with pgvector extension** or **Amazon RDS PostgreSQL with pgvector**:

```
┌────────────────────────────────────────────────────────┐
│ Vector Store Options (without OpenSearch)              │
├────────────────────────────────────────────────────────┤
│ Option 1: Aurora PostgreSQL Serverless v2 + pgvector  │
│ Option 2: RDS PostgreSQL + pgvector                    │
│ Option 3: Amazon MemoryDB for Redis (vector search)   │
│ Option 4: In-memory FAISS (for smaller datasets)      │
└────────────────────────────────────────────────────────┘
```

**Recommended: Aurora PostgreSQL Serverless v2** - scales automatically, managed, cost-effective for Gen AI workloads.

## How to Represent Shared Services

### Approach 1: Shared Services Layer (Recommended for HLD)

Create a **"Shared Services Foundation"** layer that sits beneath all blueprints:

```
┌─────────────────────────────────────────────────────────────────┐
│                     CONSUMER BLUEPRINTS                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Data Layer   │  │Model Handler │  │  Inference   │         │
│  │              │  │              │  │   Patterns   │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                 │                 │                  │
│         └─────────────────┴─────────────────┘                  │
│                           ↓                                     │
├─────────────────────────────────────────────────────────────────┤
│            SHARED SERVICES FOUNDATION                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ API Gateway  │  │  Lambda      │  │  S3 Buckets  │         │
│  │  (shared)    │  │  (shared)    │  │  (shared)    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ EventBridge  │  │     SNS      │  │     KMS      │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Secrets    │  │  CloudWatch  │  │  CloudTrail  │         │
│  │   Manager    │  │              │  │              │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Approach 2: Reference Notation (For Detailed Diagrams)

Use visual indicators to show shared services:

```
Legend:
━━━ Solid line: Direct integration
┄┄┄ Dashed line: Shared service reference
[S] Icon: Indicates shared service instance
```

Example in diagram:
```
┌─────────────────┐     ┌──────────────────┐
│  Real-Time      │────▶│ API Gateway [S]  │
│  Inference      │     └──────────────────┘
└─────────────────┘              │
                                 │
┌─────────────────┐              │
│  Async          │┄┄┄┄┄┄┄┄┄┄┄┄┄┘
│  Inference      │     (references same instance)
└─────────────────┘
```

### Approach 3: Service Catalog Pattern (My Recommendation)

Create explicit **"Shared Service Blueprints"** that other blueprints consume:

```
┌─────────────────────────────────────────────────────────────┐
│              BLUEPRINT ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │  CONSUMER BLUEPRINTS (Business Logic)              │    │
│  │  ─────────────────────────────────────────         │    │
│  │  • Data Layer                                      │    │
│  │  • Model Handler                                   │    │
│  │  • RAG Pipeline                                    │    │
│  │  • Inference Patterns (Real-time, Batch, etc.)    │    │
│  └────────────────┬───────────────────────────────────┘    │
│                   │ consumes                                │
│                   ↓                                         │
│  ┌────────────────────────────────────────────────────┐    │
│  │  SHARED SERVICE BLUEPRINTS (Infrastructure)        │    │
│  │  ───────────────────────────────────────           │    │
│  │  • API Gateway Blueprint                           │    │
│  │  • Compute Blueprint (Lambda/ECS)                  │    │
│  │  • Storage Blueprint (S3/Aurora)                   │    │
│  │  • Observability Blueprint (CloudWatch/X-Ray)      │    │
│  │  • Security Blueprint (Secrets/KMS/IAM)            │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Updated Architecture Flows

### 1. Data Lineage Flow (Updated)

```
SMUS Catalog (Producer Account - shared datasets)
    ↓ [Resource Share via RAM/Lake Formation]
AWS Glue Data Catalog (Consumer Account - local + shared refs)
    ↓
    ├─→ Athena (ad-hoc queries)
    ├─→ Glue ETL Jobs (data transformation)
    └─→ SageMaker Data Wrangler (data prep)
    ↓
S3 Buckets [Shared Service]
    ├─→ Raw Data Bucket
    ├─→ Processed Data Bucket
    ├─→ Prompts & Config Bucket ★
    └─→ Model Artifacts Bucket
    ↓
Document Processing Pipeline
    ├─→ Amazon Textract (PDF/images)
    ├─→ Lambda [Shared Service] (processing)
    └─→ Bedrock Titan Embeddings (vectorization)
    ↓
Aurora PostgreSQL + pgvector [Shared Service]
    ↓
Bedrock Knowledge Base (RAG indexing)
    ↓
Inference Endpoints
```

### 2. Model Lineage Flow (Updated)

```
Foundation Models
    ├─→ SageMaker JumpStart (Hugging Face, open models)
    └─→ Amazon Bedrock (Claude, Titan, etc.)
    ↓
Fine-tuning (if needed)
    └─→ SageMaker Training Jobs
    ↓
Evaluation & Experimentation
    ├─→ SageMaker Experiments (metrics tracking)
    ├─→ SageMaker Clarify (bias/fairness)
    └─→ Custom Evaluation (ROUGE, BLEU, human feedback)
    ↓
Model Registry
    └─→ SageMaker Model Registry (versioning, metadata)
    ↓
Approval Workflow
    └─→ Step Functions [Shared Service] (orchestration)
    ↓
Deployment
    ├─→ SageMaker Real-time Endpoint
    ├─→ SageMaker Serverless Endpoint
    ├─→ SageMaker Async Endpoint
    ├─→ SageMaker Batch Transform
    └─→ Bedrock Runtime (for foundation models)
    ↓
Monitoring
    ├─→ SageMaker Model Monitor (drift detection)
    ├─→ CloudWatch [Shared Service] (metrics/logs)
    └─→ EventBridge [Shared Service] (alerts)
```

### 3. Prompt Lineage Flow (Updated with S3)

```
Prompt Development
    ├─→ Manual creation (notebooks, IDE)
    └─→ Prompt optimization (experiments)
    ↓
S3 Prompts & Config Bucket [Shared Service]
    ├─→ /prompts/
    │   ├─→ /production/
    │   ├─→ /staging/
    │   └─→ /experiments/
    ├─→ /configs/
    │   ├─→ model_configs.json
    │   ├─→ inference_configs.json
    │   └─→ guardrails_configs.json
    └─→ /versions/ (S3 versioning enabled)
    ↓
Lambda [Shared Service] (retrieves prompts/configs)
    ↓
Prompt Assembly
    ├─→ Template rendering
    ├─→ Context injection
    └─→ Parameter substitution
    ↓
Bedrock/SageMaker Inference
    ↓
Response & Logging
    ├─→ CloudWatch Logs [Shared Service]
    └─→ S3 (inference logs)
    ↓
Evaluation & Feedback
    ├─→ Human feedback (A2I)
    ├─→ Automated metrics
    └─→ Prompt refinement (back to S3)
```

### 4. RAG Pipeline Flow (Updated without OpenSearch)

```
Document Ingestion
    ↓
S3 Raw Data Bucket [Shared Service]
    ↓
Document Processing
    ├─→ Lambda [Shared Service] (orchestration)
    ├─→ Textract (extraction)
    └─→ Step Functions [Shared Service] (workflow)
    ↓
Text Chunking & Processing
    └─→ Lambda [Shared Service] (chunking logic)
    ↓
Embeddings Generation
    ├─→ Bedrock Titan Embeddings
    └─→ SageMaker Endpoint (custom embeddings)
    ↓
Vector Storage
    └─→ Aurora PostgreSQL + pgvector [Shared Service]
        ├─→ vectors table
        ├─→ metadata table
        └─→ indexes (HNSW/IVFFlat)
    ↓
Bedrock Knowledge Base
    ├─→ Data source: Aurora PostgreSQL
    └─→ Indexing configuration
    ↓
Query Time
    ↓
API Gateway [Shared Service]
    ↓
Lambda [Shared Service] (orchestration)
    ├─→ Query Bedrock Knowledge Base (retrieval)
    ├─→ Load prompt from S3 [Shared Service]
    └─→ Query Bedrock Runtime (generation)
    ↓
Response
    ├─→ CloudWatch [Shared Service] (logging)
    └─→ Return to client
```

### 5. Inference Integration Flow (How Shared Services Connect)

```
┌──────────────────────────────────────────────────────────────┐
│                    INFERENCE REQUEST FLOW                     │
└──────────────────────────────────────────────────────────────┘

Client (GitLab/Direct/TDL)
    ↓
API Gateway [Shared Service - Single Instance]
    ├─→ WAF (DDoS protection)
    └─→ Lambda Authorizer [Shared Service] (authentication)
    ↓
Route based on endpoint path:
    │
    ├─→ /realtime  → Lambda [Shared Service] → SageMaker Real-time Endpoint
    │                    ↓
    ├─→ /async     → Lambda [Shared Service] → SageMaker Async Endpoint
    │                    ↓                      └→ SNS [Shared Service] (completion)
    │
    ├─→ /batch     → Lambda [Shared Service] → Step Functions [Shared Service]
    │                                           └→ SageMaker Batch Transform
    │
    └─→ /rag       → Lambda [Shared Service] → Bedrock Knowledge Base + Runtime
                         │
                         ├─→ S3 [Shared Service] (load prompts/configs)
                         ├─→ Aurora [Shared Service] (vector search)
                         └─→ Secrets Manager [Shared Service] (credentials)
    ↓
All inference calls log to:
    ├─→ CloudWatch Logs [Shared Service]
    ├─→ CloudWatch Metrics [Shared Service]
    └─→ X-Ray [Shared Service] (distributed tracing)
    ↓
Monitoring & Alerting:
    ├─→ EventBridge [Shared Service] (rules)
    └─→ SNS [Shared Service] (notifications)
```

### 6. End-to-End Gen AI Workflow (Complete Picture)

```
┌──────────────────────────────────────────────────────────────┐
│ PHASE 1: DATA PREPARATION                                    │
└──────────────────────────────────────────────────────────────┘
SMUS Catalog → Glue Catalog → Athena/Glue ETL → S3 [Shared]
                                                   ↓
                                          Textract + Lambda [Shared]
                                                   ↓
                                          Embeddings (Bedrock)
                                                   ↓
                                          Aurora + pgvector [Shared]

┌──────────────────────────────────────────────────────────────┐
│ PHASE 2: MODEL PREPARATION                                   │
└──────────────────────────────────────────────────────────────┘
Foundation Models (Bedrock/JumpStart)
    ↓
Optional Fine-tuning (SageMaker Training)
    ↓
Experiments + Evaluation (SageMaker Experiments, Clarify)
    ↓
Model Registry (SageMaker)
    ↓
Approval (Step Functions [Shared])
    ↓
Deployment (SageMaker Endpoints / Bedrock)

┌──────────────────────────────────────────────────────────────┐
│ PHASE 3: PROMPT & CONFIG MANAGEMENT                          │
└──────────────────────────────────────────────────────────────┘
Prompt Development
    ↓
S3 Prompts & Config Bucket [Shared]
    ├─→ Versioning enabled
    ├─→ Lifecycle policies
    └─→ Organized by environment
    ↓
Lambda [Shared] retrieves at inference time

┌──────────────────────────────────────────────────────────────┐
│ PHASE 4: INFERENCE & MONITORING                              │
└──────────────────────────────────────────────────────────────┘
API Gateway [Shared] → Lambda Authorizer [Shared]
    ↓
Lambda [Shared] (orchestration)
    ├─→ Load prompts/configs from S3 [Shared]
    ├─→ Query Aurora [Shared] (RAG context)
    ├─→ Call Bedrock/SageMaker
    └─→ Apply Bedrock Guardrails
    ↓
Response
    ↓
Logging & Monitoring
    ├─→ CloudWatch [Shared]
    ├─→ X-Ray [Shared]
    └─→ EventBridge + SNS [Shared]
    ↓
Model Monitor (drift detection)
    └─→ Triggers retraining if needed
```

## Updated Blueprint Structure

### Core Blueprints (Consumer-Specific)

1. **Data Layer Blueprint**
   - Glue Catalog (SMUS integration)
   - Athena
   - Glue ETL Jobs
   - SageMaker Data Wrangler
   - Textract integration

2. **Model Handler Blueprint**
   - SageMaker AI (training, endpoints)
   - SageMaker Experiments
   - SageMaker Clarify
   - Model Registry
   - Bedrock integration

3. **RAG Pipeline Blueprint** ★ NEW
   - Document processing flow
   - Embeddings generation
   - Aurora PostgreSQL + pgvector
   - Bedrock Knowledge Base

4. **Inference Patterns Blueprints**
   - Real-time Inference
   - Async Inference
   - Batch Inference
   - Serverless Inference

### Shared Service Blueprints (Infrastructure)

5. **API & Compute Blueprint** ★
   - API Gateway (single instance, multiple routes)
   - Lambda (multiple functions, shared runtime)
   - Lambda Authorizer
   - ECS (if needed for long-running tasks)

6. **Storage Blueprint** ★
   - S3 Buckets (organized by purpose)
     - Raw data
     - Processed data
     - Prompts & configs ★
     - Model artifacts
     - Logs & backups
   - Aurora PostgreSQL + pgvector
   - DynamoDB (conversation state, metadata)

7. **Observability Blueprint** ★
   - CloudWatch (logs, metrics, dashboards)
   - X-Ray (distributed tracing)
   - CloudTrail (audit logs)
   - EventBridge (event routing)

8. **Security Blueprint** ★
   - Secrets Manager
   - KMS (encryption keys)
   - IAM roles & policies
   - Lambda Authorizer
   - Bedrock Guardrails

9. **Orchestration Blueprint** ★
   - Step Functions (workflows)
   - EventBridge (event-driven)
   - SNS (notifications)
   - SQS (if needed for decoupling)

### Support Blueprints

10. **AI Operations Blueprint**
    - SageMaker Model Monitor
    - SageMaker Ground Truth (data labeling)
    - A2I (human review)
    - MLflow integration (optional)

11. **Data Quality Blueprint**
    - SageMaker Clarify
    - AWS Glue DataBrew
    - Data validation rules

## Visual Representation Strategy

### In Your Diagram, Use:

1. **Color Coding**
   - Blue: Consumer-specific blueprints
   - Green: Shared service blueprints
   - Orange: External integrations (SMUS, GitLab)

2. **Annotation System**
   ```
   [S] = Shared Service
   [C] = Consumer-specific
   [E] = External
   
   Example: "API Gateway [S]" or "Lambda [S]"
   ```

3. **Grouped Boxes**
   ```
   ┌─────────────────────────────────────┐
   │ Inference Patterns [C]              │
   │ ┌─────────┐ ┌─────────┐           │
   │ │Real-time│ │  Async  │           │
   │ └────┬────┘ └────┬────┘           │
   │      └───────────┴──────┐          │
   │                          ↓          │
   │              API Gateway [S]       │
   └─────────────────────────────────────┘
   ```

4. **Reference Arrows**
   - Solid arrows (━━━): Direct ownership/creation
   - Dashed arrows (┄┄┄): References shared service
   - Dotted arrows (····): Data flow

## Key Architecture Decisions Summary

| Component | Decision | Rationale |
|-----------|----------|-----------|
| Vector Store | Aurora PostgreSQL + pgvector | OpenSearch not allowed, Aurora is managed, scalable |
| Prompt Storage | S3 with versioning | Simple, cost-effective, versioned, integrates easily |
| Shared Services | Separate blueprint layer | Clear separation, reusability, easier governance |
| API Gateway | Single instance, multiple routes | Cost-efficient, centralized management |
| Monitoring | CloudWatch + X-Ray | Native AWS integration, comprehensive observability |
| RAG | Bedrock Knowledge Base + Aurora | Managed service, reduces operational overhead |
| Auth | Lambda Authorizer | Flexible, supports multiple auth patterns |

This updated architecture is production-ready for Gen AI workloads in a SageMaker Unified Studio consumer account context.


-----

import React, { useState } from 'react';
import { Database, Cloud, Cpu, Lock, Bell, FileText, GitBranch, Network, Eye, Zap, Server, Box, ArrowRight, Layers } from 'lucide-react';

const GenAIArchitecture = () => {
  const [viewMode, setViewMode] = useState('layered'); // 'layered', 'flow', 'detailed', 'security'

  return (
    <div className="w-full min-h-screen bg-gradient-to-br from-slate-50 to-slate-100 p-4">
      <div className="max-w-7xl mx-auto">
        {/* Header with View Selector */}
        <div className="bg-white rounded-lg shadow-lg p-6 mb-6">
          <h1 className="text-3xl font-bold text-gray-800 mb-2">
            Gen AI Architecture - SageMaker Unified Studio
          </h1>
          <p className="text-gray-600 mb-4">
            Consumer Linked Account | High-Level Design
          </p>
          
          {/* View Mode Toggle */}
          <div className="flex gap-2 mb-4">
            <button
              onClick={() => setViewMode('layered')}
              className={`px-4 py-2 rounded-lg font-medium transition-colors ${
                viewMode === 'layered' 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              }`}
            >
              <Layers className="inline w-4 h-4 mr-2" />
              Layered View
            </button>
            <button
              onClick={() => setViewMode('flow')}
              className={`px-4 py-2 rounded-lg font-medium transition-colors ${
                viewMode === 'flow' 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              }`}
            >
              <ArrowRight className="inline w-4 h-4 mr-2" />
              Flow View
            </button>
            <button
              onClick={() => setViewMode('detailed')}
              className={`px-4 py-2 rounded-lg font-medium transition-colors ${
                viewMode === 'detailed' 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              }`}
            >
              <Network className="inline w-4 h-4 mr-2" />
              Detailed View
            </button>
            <button
              onClick={() => setViewMode('security')}
              className={`px-4 py-2 rounded-lg font-medium transition-colors ${
                viewMode === 'security' 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              }`}
            >
              <Lock className="inline w-4 h-4 mr-2" />
              Security & IAM
            </button>
          </div>

          <div className="flex gap-4 text-sm">
            <div className="flex items-center gap-2">
              <div className="w-4 h-4 bg-blue-100 border-2 border-blue-400 rounded"></div>
              <span>Consumer Blueprint</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-4 h-4 bg-green-100 border-2 border-green-400 rounded"></div>
              <span>Shared Service [S]</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-4 h-4 bg-orange-100 border-2 border-orange-400 rounded"></div>
              <span>External [E]</span>
            </div>
          </div>
        </div>

        {/* Render based on view mode */}
        {viewMode === 'layered' && <LayeredView />}
        {viewMode === 'flow' && <FlowView />}
        {viewMode === 'detailed' && <DetailedView />}
        {viewMode === 'security' && <SecurityView />}
      </div>
    </div>
  );
};

// VIEW 1: LAYERED ARCHITECTURE (Abstract, High-Level)
const LayeredView = () => (
  <div className="space-y-6">
    <div className="bg-white rounded-lg shadow-lg p-8">
      <h2 className="text-2xl font-bold mb-6 text-center">Layered Architecture View</h2>
      
      {/* External Layer */}
      <div className="mb-8">
        <div className="bg-gradient-to-r from-orange-400 to-orange-500 text-white p-4 rounded-t-lg">
          <h3 className="font-bold text-lg flex items-center gap-2">
            <Cloud className="w-5 h-5" />
            External Integration Layer [E]
          </h3>
        </div>
        <div className="bg-orange-50 border-2 border-orange-200 rounded-b-lg p-6">
          <div className="grid grid-cols-3 gap-4">
            <ServiceBox name="SMUS Catalog" desc="Shared Data Sources" />
            <ServiceBox name="GitLab" desc="CI/CD Pipeline" />
            <ServiceBox name="On-Prem Client" desc="Direct Connect" />
          </div>
        </div>
        <div className="flex justify-center my-4">
          <div className="text-2xl text-gray-400">↓</div>
        </div>
      </div>

      {/* Application Layer */}
      <div className="mb-8">
        <div className="bg-gradient-to-r from-blue-500 to-blue-600 text-white p-4 rounded-t-lg">
          <h3 className="font-bold text-lg flex items-center gap-2">
            <Box className="w-5 h-5" />
            Application Layer - Consumer Blueprints
          </h3>
        </div>
        <div className="bg-blue-50 border-2 border-blue-200 rounded-b-lg p-6">
          <div className="grid grid-cols-2 gap-4">
            <BlueprintBox 
              name="Data Layer" 
              services={["Glue Catalog", "Athena", "Glue ETL", "Data Wrangler"]}
            />
            <BlueprintBox 
              name="RAG Pipeline" 
              services={["Document Processing", "Embeddings", "Vector Store", "Knowledge Base"]}
            />
            <BlueprintBox 
              name="Model Handler" 
              services={["SageMaker AI", "Bedrock", "Model Registry", "Experiments"]}
            />
            <BlueprintBox 
              name="Inference Patterns" 
              services={["Real-time", "Async", "Batch", "Serverless"]}
            />
          </div>
        </div>
        <div className="flex justify-center my-4">
          <div className="text-2xl text-gray-400">↓</div>
        </div>
      </div>

      {/* Infrastructure Layer */}
      <div className="mb-8">
        <div className="bg-gradient-to-r from-green-500 to-green-600 text-white p-4 rounded-t-lg">
          <h3 className="font-bold text-lg flex items-center gap-2">
            <Network className="w-5 h-5" />
            Infrastructure Layer - Shared Services [S]
          </h3>
        </div>
        <div className="bg-green-50 border-2 border-green-200 rounded-b-lg p-6">
          <div className="grid grid-cols-3 gap-4">
            <BlueprintBox 
              name="API & Compute" 
              services={["API Gateway", "Lambda", "ECS"]}
              isShared
            />
            <BlueprintBox 
              name="Storage" 
              services={["S3", "Aurora+pgvector", "DynamoDB"]}
              isShared
            />
            <BlueprintBox 
              name="Observability" 
              services={["CloudWatch", "X-Ray", "EventBridge"]}
              isShared
            />
            <BlueprintBox 
              name="Security" 
              services={["Secrets Manager", "KMS", "IAM"]}
              isShared
            />
            <BlueprintBox 
              name="Orchestration" 
              services={["Step Functions", "SNS", "SQS"]}
              isShared
            />
            <BlueprintBox 
              name="AI Operations" 
              services={["Model Monitor", "Ground Truth", "A2I"]}
              isShared
            />
          </div>
        </div>
      </div>

      {/* Foundation Layer */}
      <div>
        <div className="bg-gradient-to-r from-gray-600 to-gray-700 text-white p-4 rounded-t-lg">
          <h3 className="font-bold text-lg flex items-center gap-2">
            <Server className="w-5 h-5" />
            AWS Foundation Services
          </h3>
        </div>
        <div className="bg-gray-50 border-2 border-gray-300 rounded-b-lg p-6">
          <div className="text-center text-sm text-gray-600">
            VPC | IAM | CloudTrail | Resource Access Manager | KMS Encryption
          </div>
        </div>
      </div>
    </div>

    <ArchitectureNotes />
  </div>
);

// VIEW 2: FLOW-BASED ARCHITECTURE (Data/Request Flow)
const FlowView = () => (
  <div className="space-y-6">
    <div className="bg-white rounded-lg shadow-lg p-8">
      <h2 className="text-2xl font-bold mb-6 text-center">Flow-Based Architecture View</h2>
      
      {/* Flow 1: Data Ingestion to Inference */}
      <FlowDiagram 
        title="End-to-End Gen AI Flow"
        steps={[
          { layer: "External", items: ["SMUS Catalog", "GitLab Push"], color: "orange" },
          { layer: "Data Layer", items: ["Glue Catalog", "Athena/ETL", "S3 Storage"], color: "blue" },
          { layer: "RAG Pipeline", items: ["Document Processing", "Embeddings", "Aurora+pgvector"], color: "purple" },
          { layer: "Model Layer", items: ["Foundation Models", "Fine-tuning", "Model Registry"], color: "indigo" },
          { layer: "Inference", items: ["API Gateway [S]", "Lambda [S]", "Endpoints"], color: "cyan" },
          { layer: "Monitoring", items: ["CloudWatch [S]", "Model Monitor"], color: "green" }
        ]}
      />

      <div className="my-8 border-t-2 border-dashed border-gray-300"></div>

      {/* Flow 2: Inference Request Path */}
      <FlowDiagram 
        title="RAG Inference Request Flow"
        steps={[
          { layer: "Client", items: ["User Request"], color: "orange" },
          { layer: "API Gateway [S]", items: ["Authentication", "Rate Limiting", "Routing"], color: "green" },
          { layer: "Lambda [S]", items: ["Load Prompt from S3", "Orchestration Logic"], color: "green" },
          { layer: "Vector Search", items: ["Query Aurora [S]", "Retrieve Context"], color: "green" },
          { layer: "Knowledge Base", items: ["Bedrock KB", "Document Retrieval"], color: "purple" },
          { layer: "Generation", items: ["Bedrock Runtime", "Apply Guardrails"], color: "indigo" },
          { layer: "Response", items: ["Format Response", "Log to CloudWatch [S]"], color: "green" }
        ]}
      />
    </div>

    <ArchitectureNotes />
  </div>
);

// VIEW 3: DETAILED COMPONENT VIEW (Technical Details)
const DetailedView = () => (
  <div className="space-y-6">
    <div className="bg-white rounded-lg shadow-lg p-8">
      <h2 className="text-2xl font-bold mb-6 text-center">Detailed Component Architecture</h2>
      
      {/* Detailed Blueprint Cards */}
      <div className="space-y-6">
        <DetailedBlueprint 
          name="Data Layer Blueprint"
          color="blue"
          components={[
            {
              name: "AWS Glue Data Catalog",
              details: ["Shared catalogs from SMUS", "Local metadata tables", "Cross-account access"],
              integrations: ["→ Athena", "→ Glue ETL", "→ Data Wrangler"]
            },
            {
              name: "Amazon Athena",
              details: ["Serverless SQL queries", "Query SMUS shared data", "Result to S3 [S]"],
              integrations: ["→ S3 Storage [S]"]
            },
            {
              name: "AWS Glue ETL",
              details: ["Data transformation jobs", "PySpark/Python shells", "Scheduled/event-driven"],
              integrations: ["→ S3 [S]", "→ Aurora [S]"]
            },
            {
              name: "Amazon Textract",
              details: ["Document OCR", "Form extraction", "Table detection"],
              integrations: ["→ Lambda [S]", "→ S3 [S]"]
            }
          ]}
        />

        <DetailedBlueprint 
          name="RAG Pipeline Blueprint"
          color="purple"
          components={[
            {
              name: "Document Processing",
              details: ["S3 trigger → Lambda [S]", "Textract extraction", "Text chunking"],
              integrations: ["→ Embeddings"]
            },
            {
              name: "Embeddings Generation",
              details: ["Bedrock Titan Embeddings", "Or SageMaker custom model", "Batch processing"],
              integrations: ["→ Aurora+pgvector [S]"]
            },
            {
              name: "Aurora PostgreSQL + pgvector",
              details: ["Serverless v2", "HNSW/IVFFlat indexes", "Similarity search"],
              integrations: ["→ Bedrock Knowledge Base"]
            },
            {
              name: "Bedrock Knowledge Base",
              details: ["Managed RAG service", "Aurora data source", "Automatic sync"],
              integrations: ["→ Bedrock Runtime"]
            }
          ]}
        />

        <DetailedBlueprint 
          name="API & Compute Blueprint [S]"
          color="green"
          components={[
            {
              name: "API Gateway",
              details: ["REST/WebSocket APIs", "Multiple routes (real-time, async, batch)", "WAF integration"],
              integrations: ["→ Lambda Authorizer [S]", "→ Lambda Functions [S]"]
            },
            {
              name: "Lambda Functions",
              details: ["Inference orchestration", "Prompt loading from S3", "Vector search queries"],
              integrations: ["→ S3 [S]", "→ Aurora [S]", "→ Bedrock", "→ SageMaker"]
            },
            {
              name: "Lambda Authorizer",
              details: ["JWT validation", "API key verification", "Custom auth logic"],
              integrations: ["→ Secrets Manager [S]"]
            }
          ]}
        />

        <DetailedBlueprint 
          name="Storage Blueprint [S]"
          color="green"
          components={[
            {
              name: "S3 Buckets",
              details: [
                "Raw data bucket (landing zone)",
                "Processed data bucket",
                "Prompts & configs bucket (/prompts/, /configs/)",
                "Model artifacts bucket",
                "Logs & audit bucket"
              ],
              integrations: ["Versioning enabled", "Lifecycle policies", "Encryption: KMS [S]"]
            },
            {
              name: "Aurora PostgreSQL",
              details: ["Serverless v2 auto-scaling", "pgvector extension", "Multi-AZ"],
              integrations: ["→ VPC Private Subnets", "→ Secrets Manager [S]"]
            },
            {
              name: "DynamoDB",
              details: ["Conversation state", "Metadata storage", "Session management"],
              integrations: ["→ Lambda [S]", "→ Step Functions [S]"]
            }
          ]}
        />
      </div>
    </div>

    <ArchitectureNotes />
  </div>
);

// Helper Components
const ServiceBox = ({ name, desc }) => (
  <div className="bg-white rounded border-2 border-gray-300 p-3 text-center">
    <div className="font-semibold text-sm">{name}</div>
    <div className="text-xs text-gray-600 mt-1">{desc}</div>
  </div>
);

const BlueprintBox = ({ name, services, isShared }) => (
  <div className={`${isShared ? 'bg-white' : 'bg-white'} rounded-lg border-2 ${isShared ? 'border-green-300' : 'border-blue-300'} p-4`}>
    <div className="flex items-center justify-between mb-2">
      <h4 className="font-semibold text-sm">{name}</h4>
      {isShared && <span className="text-xs bg-green-600 text-white px-2 py-1 rounded">S</span>}
    </div>
    <ul className="space-y-1">
      {services.map((svc, idx) => (
        <li key={idx} className="text-xs text-gray-700">• {svc}</li>
      ))}
    </ul>
  </div>
);

const FlowDiagram = ({ title, steps }) => (
  <div className="mb-8">
    <h3 className="text-lg font-bold mb-4 text-gray-800">{title}</h3>
    <div className="space-y-3">
      {steps.map((step, idx) => (
        <div key={idx}>
          <div className={`bg-${step.color}-50 border-2 border-${step.color}-300 rounded-lg p-4`}>
            <div className="font-semibold text-sm mb-2 text-gray-800">{step.layer}</div>
            <div className="flex flex-wrap gap-2">
              {step.items.map((item, i) => (
                <span key={i} className="text-xs bg-white px-3 py-1 rounded-full border border-gray-300">
                  {item}
                </span>
              ))}
            </div>
          </div>
          {idx < steps.length - 1 && (
            <div className="flex justify-center my-2">
              <div className="text-xl text-gray-400">↓</div>
            </div>
          )}
        </div>
      ))}
    </div>
  </div>
);

const DetailedBlueprint = ({ name, color, components }) => (
  <div className="border-2 border-gray-300 rounded-lg overflow-hidden">
    <div className={`bg-${color}-500 text-white p-4`}>
      <h3 className="font-bold text-lg">{name}</h3>
    </div>
    <div className="p-6 bg-gray-50">
      <div className="space-y-4">
        {components.map((comp, idx) => (
          <div key={idx} className="bg-white rounded-lg border border-gray-200 p-4">
            <h4 className="font-semibold text-sm mb-2 text-gray-800">{comp.name}</h4>
            <div className="grid grid-cols-2 gap-4">
              <div>
                <div className="text-xs font-medium text-gray-600 mb-1">Details:</div>
                <ul className="space-y-1">
                  {comp.details.map((detail, i) => (
                    <li key={i} className="text-xs text-gray-700">• {detail}</li>
                  ))}
                </ul>
              </div>
              <div>
                <div className="text-xs font-medium text-gray-600 mb-1">Integrations:</div>
                <div className="space-y-1">
                  {comp.integrations.map((int, i) => (
                    <div key={i} className="text-xs text-blue-600">{int}</div>
                  ))}
                </div>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  </div>
);

const ArchitectureNotes = () => (
  <div className="grid grid-cols-2 gap-4">
    <div className="bg-white rounded-lg shadow-lg p-6">
      <h3 className="font-bold text-lg mb-3">Key Principles</h3>
      <ul className="space-y-2 text-sm">
        <li className="flex items-start gap-2">
          <span className="text-green-600 mt-1">✓</span>
          <span><strong>Shared Services:</strong> Single instance used by multiple blueprints</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-green-600 mt-1">✓</span>
          <span><strong>Separation of Concerns:</strong> Business logic separate from infrastructure</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-green-600 mt-1">✓</span>
          <span><strong>Scalability:</strong> Serverless and auto-scaling components</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-green-600 mt-1">✓</span>
          <span><strong>Observability:</strong> Centralized logging and monitoring</span>
        </li>
      </ul>
    </div>
    <div className="bg-white rounded-lg shadow-lg p-6">
      <h3 className="font-bold text-lg mb-3">Critical Decisions</h3>
      <ul className="space-y-2 text-sm">
        <li className="flex items-start gap-2">
          <span className="text-blue-600 mt-1">→</span>
          <span><strong>Vector Store:</strong> Aurora+pgvector (OpenSearch not allowed)</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-blue-600 mt-1">→</span>
          <span><strong>Prompts:</strong> S3 with versioning for config management</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-blue-600 mt-1">→</span>
          <span><strong>Data Access:</strong> SMUS catalog via Glue cross-account</span>
        </li>
        <li className="flex items-start gap-2">
          <span className="text-blue-600 mt-1">→</span>
          <span><strong>Gen AI:</strong> Bedrock for managed foundation models</span>
        </li>
      </ul>
    </div>
  </div>
);

// VIEW 4: SECURITY & IAM ARCHITECTURE
const SecurityView = () => (
  <div className="space-y-6">
    <div className="bg-white rounded-lg shadow-lg p-8">
      <h2 className="text-2xl font-bold mb-6 text-center">Security & IAM Architecture</h2>

      {/* IAM Role Structure */}
      <div className="mb-8">
        <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
          <Lock className="w-6 h-6 text-red-600" />
          IAM Role Architecture
        </h3>
        
        <div className="grid grid-cols-2 gap-6">
          {/* Service Roles */}
          <div className="space-y-4">
            <h4 className="font-semibold text-lg text-gray-800 mb-3">Service Execution Roles</h4>
            
            <IAMRoleCard
              roleName="SageMakerExecutionRole"
              purpose="SageMaker Studio, Training, Endpoints"
              permissions={[
                "s3:GetObject, PutObject (data/model buckets)",
                "sagemaker:* (full SageMaker access)",
                "ecr:GetAuthorizationToken, BatchGetImage",
                "logs:CreateLogGroup, CreateLogStream",
                "cloudwatch:PutMetricData",
                "kms:Decrypt, GenerateDataKey"
              ]}
              trustPolicy="sagemaker.amazonaws.com"
            />

            <IAMRoleCard
              roleName="LambdaInferenceRole"
              purpose="Lambda functions for inference orchestration"
              permissions={[
                "s3:GetObject (prompts/configs bucket)",
                "bedrock:InvokeModel, InvokeModelWithResponseStream",
                "sagemaker:InvokeEndpoint",
                "rds-data:ExecuteStatement (Aurora queries)",
                "secretsmanager:GetSecretValue",
                "logs:CreateLogStream, PutLogEvents",
                "xray:PutTraceSegments"
              ]}
              trustPolicy="lambda.amazonaws.com"
            />

            <IAMRoleCard
              roleName="GlueETLRole"
              purpose="Glue ETL jobs and crawlers"
              permissions={[
                "s3:GetObject, PutObject (data buckets)",
                "glue:GetDatabase, GetTable, GetPartitions",
                "logs:CreateLogGroup, PutLogEvents",
                "kms:Decrypt, GenerateDataKey"
              ]}
              trustPolicy="glue.amazonaws.com"
            />

            <IAMRoleCard
              roleName="StepFunctionsExecutionRole"
              purpose="Workflow orchestration"
              permissions={[
                "lambda:InvokeFunction",
                "sagemaker:CreateTrainingJob, CreateEndpoint",
                "sns:Publish",
                "events:PutTargets, PutRule",
                "logs:CreateLogGroup, PutLogEvents"
              ]}
              trustPolicy="states.amazonaws.com"
            />
          </div>

          {/* User/Application Roles */}
          <div className="space-y-4">
            <h4 className="font-semibold text-lg text-gray-800 mb-3">User & Application Roles</h4>
            
            <IAMRoleCard
              roleName="DataScientistRole"
              purpose="SageMaker Studio users"
              permissions={[
                "sagemaker:CreateNotebookInstance, CreateTrainingJob",
                "sagemaker:InvokeEndpoint (dev/test endpoints)",
                "s3:GetObject, PutObject (user workspace)",
                "glue:GetTable, GetDatabase (read-only)",
                "bedrock:InvokeModel (limited models)",
                "sts:AssumeRole (SageMakerExecutionRole)"
              ]}
              trustPolicy="SSO/Federated identity"
            />

            <IAMRoleCard
              roleName="MLOpsRole"
              purpose="CI/CD and deployment automation"
              permissions={[
                "sagemaker:CreateModel, CreateEndpointConfig",
                "sagemaker:UpdateEndpoint, DeleteEndpoint",
                "codebuild:StartBuild",
                "codepipeline:PutJobSuccessResult",
                "s3:PutObject (model registry bucket)",
                "iam:PassRole (service roles)"
              ]}
              trustPolicy="codebuild.amazonaws.com, codepipeline.amazonaws.com"
            />

            <IAMRoleCard
              roleName="BedrockKnowledgeBaseRole"
              purpose="Bedrock Knowledge Base access"
              permissions={[
                "rds-data:ExecuteStatement (Aurora data source)",
                "s3:GetObject (document buckets)",
                "bedrock:InvokeModel (embedding models)",
                "secretsmanager:GetSecretValue"
              ]}
              trustPolicy="bedrock.amazonaws.com"
            />

            <IAMRoleCard
              roleName="APIGatewayInvocationRole"
              purpose="API Gateway to invoke backend services"
              permissions={[
                "lambda:InvokeFunction",
                "sagemaker:InvokeEndpoint",
                "logs:CreateLogGroup, PutLogEvents"
              ]}
              trustPolicy="apigateway.amazonaws.com"
            />
          </div>
        </div>
      </div>

      {/* Resource-Based Policies */}
      <div className="mb-8">
        <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
          <Lock className="w-6 h-6 text-blue-600" />
          Resource-Based Policies
        </h3>

        <div className="grid grid-cols-2 gap-6">
          <ResourcePolicyCard
            resourceType="S3 Bucket Policies"
            resources={[
              {
                name: "s3://genai-data-bucket",
                policy: [
                  "Allow: SageMakerExecutionRole → GetObject, PutObject",
                  "Allow: GlueETLRole → GetObject, PutObject",
                  "Deny: All → DeleteBucket (explicit)",
                  "Require: aws:SecureTransport (HTTPS only)",
                  "Require: s3:x-amz-server-side-encryption"
                ]
              },
              {
                name: "s3://genai-prompts-configs",
                policy: [
                  "Allow: LambdaInferenceRole → GetObject",
                  "Allow: DataScientistRole → GetObject, PutObject",
                  "Versioning: Enabled",
                  "MFA Delete: Required for production"
                ]
              }
            ]}
          />

          <ResourcePolicyCard
            resourceType="KMS Key Policies"
            resources={[
              {
                name: "GenAI-Data-Key",
                policy: [
                  "Key Admins: MLOpsRole, SecurityAdminRole",
                  "Key Users: SageMakerExecutionRole, LambdaInferenceRole, GlueETLRole",
                  "Grant: kms:Decrypt, GenerateDataKey",
                  "Key Rotation: Enabled (annual)"
                ]
              },
              {
                name: "GenAI-Model-Key",
                policy: [
                  "Key Users: SageMakerExecutionRole, LambdaInferenceRole",
                  "Grant: kms:Decrypt (inference only)",
                  "Condition: aws:PrincipalOrgID = <org-id>"
                ]
              }
            ]}
          />

          <ResourcePolicyCard
            resourceType="Secrets Manager"
            resources={[
              {
                name: "aurora-db-credentials",
                policy: [
                  "Allow: LambdaInferenceRole, BedrockKBRole → GetSecretValue",
                  "Rotation: Lambda function (30 days)",
                  "Encryption: GenAI-Data-Key"
                ]
              },
              {
                name: "api-keys",
                policy: [
                  "Allow: LambdaAuthorizerRole → GetSecretValue",
                  "Deny: * → DeleteSecret (explicit)",
                  "Encryption: GenAI-Data-Key"
                ]
              }
            ]}
          />

          <ResourcePolicyCard
            resourceType="Glue Data Catalog"
            resources={[
              {
                name: "Database: smus_shared",
                policy: [
                  "Resource Policy: Allow cross-account from SMUS producer",
                  "Principal: arn:aws:iam::PRODUCER_ACCOUNT:root",
                  "Permissions: glue:GetDatabase, GetTable (read-only)",
                  "Condition: aws:PrincipalOrgID = <org-id>"
                ]
              },
              {
                name: "Database: genai_consumer",
                policy: [
                  "Allow: GlueETLRole → ALL operations",
                  "Allow: DataScientistRole → GetTable, GetPartitions",
                  "Allow: SageMakerExecutionRole → GetTable"
                ]
              }
            ]}
          />
        </div>
      </div>

      {/* Cross-Account Access */}
      <div className="mb-8">
        <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
          <Network className="w-6 h-6 text-purple-600" />
          Cross-Account Access (SMUS Integration)
        </h3>

        <div className="bg-purple-50 border-2 border-purple-300 rounded-lg p-6">
          <div className="space-y-4">
            <CrossAccountFlow
              title="SMUS Producer → Consumer Account"
              steps={[
                {
                  account: "SMUS Producer Account",
                  actions: [
                    "1. Create Resource Share in AWS RAM",
                    "2. Share Glue Database: 'shared-datasets'",
                    "3. Grant permissions: glue:GetDatabase, GetTable",
                    "4. Share with Consumer Account ID or Organization"
                  ]
                },
                {
                  account: "Consumer Account (This Account)",
                  actions: [
                    "1. Accept Resource Share in AWS RAM",
                    "2. Create Glue Catalog reference to shared database",
                    "3. Create IAM policies for roles to access shared data",
                    "4. Use in Athena/Glue ETL: FROM shared-datasets.table"
                  ]
                }
              ]}
            />

            <CrossAccountFlow
              title="Consumer → SMUS (Optional Share Back)"
              steps={[
                {
                  account: "Consumer Account (This Account)",
                  actions: [
                    "1. Enable Lake Formation (if sharing governed tables)",
                    "2. Create Resource Share for consumer-generated data",
                    "3. Grant SELECT permissions to SMUS account",
                    "4. Tag resources with data classification"
                  ]
                },
                {
                  account: "SMUS Producer Account",
                  actions: [
                    "1. Accept Resource Share",
                    "2. Create IAM policies for SMUS users",
                    "3. Access via Athena/Glue in SMUS catalog"
                  ]
                }
              ]}
            />
          </div>
        </div>
      </div>

      {/* Security Best Practices */}
      <div className="mb-8">
        <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
          <Lock className="w-6 h-6 text-green-600" />
          Security Best Practices
        </h3>

        <div className="grid grid-cols-3 gap-4">
          <BestPracticeCard
            title="Least Privilege"
            practices={[
              "Grant minimum permissions required",
              "Use resource-level permissions",
              "Implement permission boundaries",
              "Regular access reviews (quarterly)"
            ]}
          />
          <BestPracticeCard
            title="Encryption"
            practices={[
              "Encrypt at rest: S3, Aurora, EBS (KMS)",
              "Encrypt in transit: TLS 1.2+",
              "Separate KMS keys by environment",
              "Enable key rotation"
            ]}
          />
          <BestPracticeCard
            title="Monitoring & Audit"
            practices={[
              "CloudTrail: All API calls logged",
              "GuardDuty: Threat detection",
              "Config: Resource compliance",
              "Security Hub: Centralized findings"
            ]}
          />
          <BestPracticeCard
            title="Network Security"
            practices={[
              "VPC: Private subnets for compute",
              "Security Groups: Least access",
              "VPC Endpoints: Private AWS service access",
              "WAF: API Gateway protection"
            ]}
          />
          <BestPracticeCard
            title="Identity"
            practices={[
              "No long-term credentials",
              "Use IAM roles for service access",
              "MFA for admin operations",
              "Rotate secrets regularly"
            ]}
          />
          <BestPracticeCard
            title="Data Protection"
            practices={[
              "S3: Versioning + MFA Delete",
              "Bedrock: Guardrails for content filtering",
              "DLP: PII detection in logs",
              "Backup: Automated snapshots"
            ]}
          />
        </div>
      </div>

      {/* Security Architecture Diagram */}
      <div>
        <h3 className="text-xl font-bold mb-4">Security Flow Example: Inference Request</h3>
        <SecurityFlowDiagram />
      </div>
    </div>
  </div>
);

// Helper Components for Security View
const IAMRoleCard = ({ roleName, purpose, permissions, trustPolicy }) => (
  <div className="bg-white border-2 border-gray-300 rounded-lg p-4 hover:shadow-md transition-shadow">
    <div className="flex items-start justify-between mb-2">
      <h5 className="font-bold text-sm text-gray-800">{roleName}</h5>
      <span className="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded">Role</span>
    </div>
    <p className="text-xs text-gray-600 mb-3 italic">{purpose}</p>
    <div className="mb-3">
      <div className="text-xs font-semibold text-gray-700 mb-1">Trust Policy:</div>
      <div className="text-xs bg-gray-100 px-2 py-1 rounded">{trustPolicy}</div>
    </div>
    <div>
      <div className="text-xs font-semibold text-gray-700 mb-1">Key Permissions:</div>
      <ul className="space-y-1">
        {permissions.map((perm, idx) => (
          <li key={idx} className="text-xs text-gray-600 flex items-start gap-1">
            <span className="text-green-600 mt-0.5">•</span>
            <span>{perm}</span>
          </li>
        ))}
      </ul>
    </div>
  </div>
);

const ResourcePolicyCard = ({ resourceType, resources }) => (
  <div className="bg-white border-2 border-blue-300 rounded-lg p-4">
    <h5 className="font-bold text-sm text-gray-800 mb-3">{resourceType}</h5>
    <div className="space-y-3">
      {resources.map((resource, idx) => (
        <div key={idx} className="bg-blue-50 rounded p-3">
          <div className="font-semibold text-xs text-blue-900 mb-2">{resource.name}</div>
          <ul className="space-y-1">
            {resource.policy.map((item, i) => (
              <li key={i} className="text-xs text-gray-700">• {item}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  </div>
);

const CrossAccountFlow = ({ title, steps }) => (
  <div className="bg-white border-2 border-purple-300 rounded-lg p-4">
    <h5 className="font-bold text-sm text-purple-900 mb-3">{title}</h5>
    <div className="space-y-4">
      {steps.map((step, idx) => (
        <div key={idx}>
          <div className="font-semibold text-xs text-gray-800 mb-2 flex items-center gap-2">
            <span className="bg-purple-600 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs">
              {idx + 1}
            </span>
            {step.account}
          </div>
          <ul className="ml-7 space-y-1">
            {step.actions.map((action, i) => (
              <li key={i} className="text-xs text-gray-700">{action}</li>
            ))}
          </ul>
          {idx < steps.length - 1 && (
            <div className="flex justify-center my-2">
              <div className="text-purple-400">↓</div>
            </div>
          )}
        </div>
      ))}
    </div>
  </div>
);

const BestPracticeCard = ({ title, practices }) => (
  <div className="bg-gradient-to-br from-green-50 to-green-100 border-2 border-green-300 rounded-lg p-4">
    <h5 className="font-bold text-sm text-green-900 mb-2">{title}</h5>
    <ul className="space-y-1">
      {practices.map((practice, idx) => (
        <li key={idx} className="text-xs text-gray-700 flex items-start gap-1">
          <span className="text-green-600 mt-0.5">✓</span>
          <span>{practice}</span>
        </li>
      ))}
    </ul>
  </div>
);

const SecurityFlowDiagram = () => (
  <div className="bg-gradient-to-br from-slate-50 to-slate-100 border-2 border-slate-300 rounded-lg p-6">
    <div className="space-y-3">
      <SecurityFlowStep
        step="1"
        actor="Client (GitLab/On-Prem)"
        action="HTTP Request with API Key"
        security="TLS 1.2+ encryption"
      />
      <SecurityFlowStep
        step="2"
        actor="API Gateway + WAF"
        action="Rate limiting, DDoS protection, request validation"
        security="AWS WAF rules, throttling"
      />
      <SecurityFlowStep
        step="3"
        actor="Lambda Authorizer"
        action="Validate API key from Secrets Manager"
        security="IAM role: LambdaAuthorizerRole"
      />
      <SecurityFlowStep
        step="4"
        actor="Lambda Inference Function"
        action="Load prompts from S3, query Aurora, call Bedrock"
        security="IAM role: LambdaInferenceRole | KMS decrypt | VPC private subnet"
      />
      <SecurityFlowStep
        step="5"
        actor="Bedrock + Guardrails"
        action="Generate response with content filtering"
        security="Bedrock Guardrails: PII filtering, harmful content blocking"
      />
      <SecurityFlowStep
        step="6"
        actor="CloudWatch + CloudTrail"
        action="Log request/response, audit API calls"
        security="Encrypted logs (KMS) | 90-day retention"
      />
    </div>
  </div>
);

const SecurityFlowStep = ({ step, actor, action, security }) => (
  <div className="bg-white rounded-lg border-2 border-gray-300 p-4">
    <div className="flex items-start gap-3">
      <div className="bg-blue-600 text-white rounded-full w-8 h-8 flex items-center justify-center font-bold flex-shrink-0">
        {step}
      </div>
      <div className="flex-1">
        <div className="font-semibold text-sm text-gray-800 mb-1">{actor}</div>
        <div className="text-xs text-gray-600 mb-2">{action}</div>
        <div className="flex items-center gap-2">
          <Lock className="w-3 h-3 text-green-600" />
          <div className="text-xs text-green-700 font-medium">{security}</div>
        </div>
      </div>
    </div>
  </div>
);

export default GenAIArchitecture;


## New Security & IAM Architecture View

### 1. **IAM Role Architecture**

**Service Execution Roles:**
- **SageMakerExecutionRole** - For Studio, training jobs, and endpoints
- **LambdaInferenceRole** - For inference orchestration with Bedrock/SageMaker
- **GlueETLRole** - For data transformation jobs
- **StepFunctionsExecutionRole** - For workflow orchestration

**User & Application Roles:**
- **DataScientistRole** - For SageMaker Studio users
- **MLOpsRole** - For CI/CD and deployment automation
- **BedrockKnowledgeBaseRole** - For Bedrock KB to access Aurora
- **APIGatewayInvocationRole** - For API Gateway to invoke backends

Each role shows:
- Purpose and use case
- Key permissions (specific actions)
- Trust policy (which service can assume it)

### 2. **Resource-Based Policies**

Covers policies for:
- **S3 Buckets** (data, prompts/configs) - Including encryption requirements, MFA delete
- **KMS Keys** (data encryption, model encryption) - Key admins, users, rotation
- **Secrets Manager** (DB credentials, API keys) - Access control, rotation policies
- **Glue Data Catalog** (cross-account access for SMUS) - Resource policies for sharing

### 3. **Cross-Account Access (SMUS Integration)**

Detailed flows for:
- **Producer → Consumer**: How SMUS shares data catalogs via AWS RAM
- **Consumer → Producer** (optional): How to share back to SMUS using Lake Formation

Shows step-by-step actions in each account.

### 4. **Security Best Practices**

Six key areas:
- **Least Privilege** - Minimum permissions, regular reviews
- **Encryption** - At rest and in transit requirements
- **Monitoring & Audit** - CloudTrail, GuardDuty, Security Hub
- **Network Security** - VPC, security groups, endpoints
- **Identity** - No long-term credentials, IAM roles
- **Data Protection** - Versioning, guardrails, backups

### 5. **Security Flow Example**

Step-by-step security flow for an inference request showing:
1. TLS encryption from client
2. WAF protection at API Gateway
3. Lambda Authorizer validation
4. IAM role-based access to resources
5. Bedrock Guardrails for content filtering
6. CloudWatch/CloudTrail audit logging

## How to Represent Security in Your Original Diagram

For your box-and-arrow HLD, I recommend adding:

1. **Security overlay** - Dotted box around services showing:
   ```
   ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
   │  Security Boundary (VPC)              │
   │  ┌──────────┐  ┌──────────┐          │
   │  │ Lambda   │  │ Aurora   │          │
   │  └──────────┘  └──────────┘          │
   └─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
   ```

2. **IAM role annotations** on arrows:
   ```
   Lambda ─[LambdaInferenceRole]→ Bedrock
   Lambda ─[LambdaInferenceRole]→ S3
   ```

3. **Security components as a layer**:
   ```
   ┌────────────────────────────────────┐
   │  Security Services                 │
   │  KMS | Secrets Mgr | IAM | WAF    │
   └────────────────────────────────────┘
   ```

4. **Add security legend**:
   ```
   🔒 Encrypted at rest (KMS)
   🔐 Encrypted in transit (TLS)
   👤 IAM role required
   🛡️ Resource policy applied
   ```

The interactive diagram now has all four views - you can toggle between them to see the architecture from different perspectives!