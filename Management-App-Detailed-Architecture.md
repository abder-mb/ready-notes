# Management App - Detailed Technical Architecture

## Document Information
- **Version**: 1.0
- **Date**: January 14, 2026
- **Architecture**: AWS-based Data Mesh Management Platform
- **Technology Stack**: EKS, Python FastAPI, AWS DataZone, Glue, Lake Formation, EventBridge, Step Functions

---

## Table of Contents

1. [Overview](#overview)
2. [Layer 1: User Interaction Layer](#layer-1-user-interaction-layer)
3. [Layer 2: Application Layer](#layer-2-application-layer)
4. [Layer 3: Orchestration & Event Layer](#layer-3-orchestration--event-layer)
5. [Layer 4: AWS Service Integration](#layer-4-aws-service-integration)
6. [Layer 5: Data Persistence](#layer-5-data-persistence)
7. [Data Flows & Sequences](#data-flows--sequences)
8. [Security Architecture](#security-architecture)
9. [Network Architecture](#network-architecture)
10. [Infrastructure & Deployment](#infrastructure--deployment)
11. [Monitoring & Operations](#monitoring--operations)
12. [Scalability & Performance](#scalability--performance)
13. [Disaster Recovery](#disaster-recovery)
14. [Cost Estimation](#cost-estimation)

---

## Overview

The Management App is a centralized control plane for a Data Mesh ecosystem built on AWS. It enables domain teams to:

- Create and manage data domains and projects
- Publish data assets with metadata
- Control access permissions (Lake Formation)
- Request and approve data subscriptions
- Share data across AWS accounts (RAM + Lake Formation)
- Track data lineage and quality
- Deploy infrastructure (CloudFormation templates)

### Architecture Principles

1. **Microservices**: 12 independent services, each with single responsibility
2. **Event-Driven**: Async communication via EventBridge
3. **Workflow Orchestration**: Step Functions for multi-step processes
4. **Infrastructure as Code**: CloudFormation + Terraform
5. **Private by Default**: No public internet exposure
6. **Serverless-First**: Minimal infrastructure management

---

## Layer 1: User Interaction Layer

### Components

#### 1.1 Route 53 (DNS)
**Purpose**: Domain name resolution

**Configuration**:
- **Hosted Zone**: Private hosted zone (optional) or Public (if accessible from internet)
- **Record Type**: CNAME or Alias
- **Target**: API Gateway endpoint
- **Example**: `datamesh-api.company.com` → `xyz123.execute-api.us-east-1.amazonaws.com`

**High-Level Details**:
```
Client Browser (On-Prem)
    │
    ├─ DNS Query: "What is the IP of datamesh-api.company.com?"
    │
    ▼
Route 53 (DNS Server)
    │
    ├─ Lookup record
    ├─ Return: 52.94.12.34 (API Gateway IP)
    │
    ▼
Client Browser
    └─ Caches response (TTL: 60 seconds)
    └─ Next requests go directly to API Gateway IP
```

**DNS Configuration**:
```
Name: datamesh-api.company.com
Type: CNAME
Value: xyz123.execute-api.us-east-1.amazonaws.com
TTL: 300 seconds
```

---

#### 1.2 API Gateway (Regional)
**Purpose**: Public entry point for HTTP API requests

**Configuration**:
- **Type**: HTTP API (v2) for better performance than REST API
- **Endpoint Type**: Regional
- **Stage**: prod, staging, dev
- **Throttling**: 10,000 requests/second (account limit)
- **Rate Limiting**: Custom via Lambda Authorizer

**API Endpoints**:
```
POST   /api/v1/domains              - Create domain
GET    /api/v1/domains              - List domains
POST   /api/v1/projects             - Create project
POST   /api/v1/assets               - Publish asset
POST   /api/v1/subscriptions        - Request subscription
POST   /api/v1/permissions/grant    - Grant permission
GET    /api/v1/lineage/{assetId}   - Get asset lineage
```

**Integration with VPC Link**:
```
API Gateway
    │
    ├─ Resource: /api/v1/
    ├─ Method: POST, GET
    │
    └─ Integration Type: VPC_LINK
        ├─ VPC Link ID: vlnk-abc123xyz
        ├─ Target: http://internal-alb.vpc.local
        ├─ Port: 80
        └─ Path: {proxy+}  (forward all requests)
```

---

#### 1.3 AWS WAF (Web Application Firewall)
**Purpose**: Protect API Gateway from malicious requests

**Configuration**:
- **Scope**: REGIONAL
- **Rules**:
  - Rate limiting: 2000 requests/5 minutes per IP
  - SQL injection protection (OWASP rule group)
  - XSS attack protection
  - Geographic restrictions (allow only company regions)
  - Bot control (optional)

**Rule Example**:
```
Rule: RateLimitRule
├─ Type: RATE_BASED
├─ Limit: 2000 requests per 5 minutes
├─ Scope: Single IP address
└─ Action: Block (return 403)

Rule: SQLInjectionProtection
├─ Type: MANAGED_RULE_GROUP
├─ Vendor: AWS
├─ Name: AWSManagedRulesSQLiRuleSet
└─ Action: Block
```

---

#### 1.4 Lambda Authorizer
**Purpose**: Validate API requests and enforce access policies

**Function Configuration**:
- **Runtime**: Python 3.11
- **Timeout**: 30 seconds
- **Memory**: 256 MB
- **Execution Role**: `LambdaAuthorizerExecutionRole`

**Authentication Flow**:
```
API Gateway receives request
    │
    ├─ Extract Authorization header
    │   Example: "Authorization: Bearer eyJhbGc..."
    │
    ▼
Invoke Lambda Authorizer
    │
    ├─ Token extraction
    ├─ JWT signature validation (verify with Cognito)
    ├─ Token expiration check
    ├─ User group/role validation
    │
    ▼
Return IAM Policy
    │
    ├─ If valid:
    │   {
    │     "principalId": "user@company.com",
    │     "policyDocument": {
    │       "Version": "2012-10-17",
    │       "Statement": [{
    │         "Action": "execute-api:Invoke",
    │         "Effect": "Allow",
    │         "Resource": "arn:aws:execute-api:us-east-1:123456789012:xyz123/prod/*"
    │       }]
    │     },
    │     "context": {
    │       "userId": "user123",
    │       "email": "user@company.com",
    │       "groups": ["admins", "finance-domain"]
    │     }
    │   }
    │
    └─ If invalid: Return Deny policy (403 Forbidden)
```

**Python Code Snippet**:
```python
import json
import boto3
from jose import jwt

cognito = boto3.client('cognito-idp')

def lambda_handler(event, context):
    token = event['authorizationToken']
    
    try:
        # Validate JWT signature
        payload = jwt.get_unverified_claims(token)
        
        # Verify token with Cognito
        # ... validation logic ...
        
        # Extract user info
        user_id = payload['sub']
        email = payload['email']
        groups = payload.get('cognito:groups', [])
        
        # Generate policy
        policy = {
            'principalId': user_id,
            'policyDocument': {
                'Version': '2012-10-17',
                'Statement': [{
                    'Action': 'execute-api:Invoke',
                    'Effect': 'Allow',
                    'Resource': '*'
                }]
            },
            'context': {
                'userId': user_id,
                'email': email,
                'groups': ','.join(groups)
            }
        }
        
        return policy
        
    except Exception as e:
        print(f"Unauthorized: {str(e)}")
        raise Exception('Unauthorized')
```

---

#### 1.5 VPC Link
**Purpose**: Private integration between API Gateway and internal ALB

**Configuration**:
- **Type**: Private Link (HTTP API)
- **Target Type**: Network Load Balancer (NLB) or Application Load Balancer (ALB)
- **Subnets**: Private subnets (10.0.1.0/24, 10.0.2.0/24)
- **Security Groups**: Allow inbound on port 80/443 from API Gateway

**VPC Link ENIs**:
```
VPC Link ID: vlnk-abc123xyz
│
├─ ENI 1: 10.0.1.50 (Private Subnet 1, AZ1)
├─ ENI 2: 10.0.2.50 (Private Subnet 2, AZ2)
│
└─ Provides private connection from API Gateway to ALB
   (no public IP, no internet gateway needed)
```

**Request Flow**:
```
Client (On-Prem)
    │
    ├─ HTTPS Request to: https://datamesh-api.company.com/api/v1/projects
    │
    ▼
API Gateway
    │
    ├─ WAF: Block malicious requests
    ├─ Lambda Authorizer: Validate JWT token
    ├─ VPC Link: Route to internal ALB
    │
    ▼
VPC Link ENIs (in Private Subnets)
    │
    ├─ PrivateLink tunnel (AWS backbone)
    │
    ▼
ALB (internal-alb.vpc.local)
    │
    ├─ Port 80 (HTTP)
    ├─ Listener Rule: /api/v1/* → Target Group
    │
    ▼
Target Group (EKS Pods)
    │
    └─ Route to: project-service:8001
```

---

## Layer 2: Application Layer

### EKS Cluster Configuration

**Cluster Details**:
- **Name**: management-cluster
- **Kubernetes Version**: 1.28+
- **Node Type**: EC2 (t3.xlarge, 4 vCPU, 16 GB RAM) or Fargate
- **Worker Nodes**: 3-10 nodes (auto-scaling)
- **Availability Zones**: 2+ (for HA)

**Networking**:
- **VPC**: 10.0.0.0/16
- **Private Subnets**: 10.0.1.0/24 (AZ1), 10.0.2.0/24 (AZ2)
- **ENI per node**: Multiple pods per node
- **Pod Network**: AWS VPC CNI (native subnets)

### Microservices Architecture

#### 2.1 Service 1: Domain Service
**Purpose**: Manage DataZone domains

**Configuration**:
- **Language**: Python 3.11
- **Framework**: FastAPI
- **Port**: 8000
- **Replicas**: 3-5 (HPA: CPU > 70%)
- **Image**: `domain-service:v1.0.0`

**Responsibilities**:
- Create domains in DataZone
- Update domain settings
- List domains with filters
- Delete domains

**Kubernetes Resources**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: domain-service-sa
  namespace: management-app
  annotations:
    eks.amazonaws.com/role-arn: 
      arn:aws:iam::123456789012:role/DomainServiceRole

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: domain-service
  namespace: management-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: domain-service
  template:
    metadata:
      labels:
        app: domain-service
    spec:
      serviceAccountName: domain-service-sa
      containers:
      - name: domain-svc
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/domain-service:v1.0.0
        ports:
        - containerPort: 8000
        env:
        - name: AWS_REGION
          value: us-east-1
        - name: ENVIRONMENT
          value: prod
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: domain-svc
  namespace: management-app
spec:
  type: ClusterIP
  selector:
    app: domain-service
  ports:
  - port: 8000
    targetPort: 8000
```

**IAM Role (IRSA)**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "datazone:CreateDomain",
        "datazone:UpdateDomain",
        "datazone:DescribeDomain",
        "datazone:ListDomains",
        "datazone:DeleteDomain"
      ],
      "Resource": "arn:aws:datazone:us-east-1:123456789012:domain/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:us-east-1:123456789012:log-group:/aws/eks/*"
    }
  ]
}
```

#### 2.2 Service 2: Project Service
**Purpose**: Manage data projects within domains

**Port**: 8001 | **Replicas**: 5 | **IRSA Role**: ProjectServiceRole

**API Endpoints**:
```
POST   /api/projects              - Create project
GET    /api/projects              - List projects (with pagination)
GET    /api/projects/{projectId}  - Get project details
POST   /api/projects/{projectId}  - Update project
DELETE /api/projects/{projectId}  - Delete project
POST   /api/projects/{projectId}/promote  - Promote to next environment
```

**Database Calls** (via boto3):
```python
import boto3

datazone = boto3.client('datazone')

def create_project(domain_id, project_name):
    response = datazone.create_project(
        domainIdentifier=domain_id,
        name=project_name,
        description='Auto-created project'
    )
    
    # Publish event to EventBridge
    eventbridge = boto3.client('events')
    eventbridge.put_events(
        Entries=[{
            'Source': 'management-app.projects',
            'DetailType': 'ProjectCreated',
            'Detail': json.dumps({
                'projectId': response['id'],
                'projectName': project_name,
                'domainId': domain_id,
                'createdAt': datetime.utcnow().isoformat()
            }),
            'EventBusName': 'datamesh-events'
        }]
    )
    
    return response
```

#### 2.3 Service 3: Asset Service
**Purpose**: Publish and manage data assets

**Port**: 8002 | **Replicas**: 4 | **IRSA Role**: AssetServiceRole

**Workflow**:
```
1. User publishes asset (S3 location, Glue table, Redshift table)
2. Asset Service:
   - Validates metadata
   - Registers with Glue Catalog
   - Publishes to DataZone
   - Creates Iceberg snapshot (if applicable)
3. Publish event: AssetPublished
4. EventBridge triggers downstream workflows
```

#### 2.4 Service 4: Catalog Service
**Purpose**: Manage DataZone glossary and metadata forms

**Port**: 8003 | **Replicas**: 2 | **IRSA Role**: CatalogServiceRole

**Responsibilities**:
- Create/update glossary terms
- Define metadata forms
- Link metadata to assets
- Search capabilities

#### 2.5 Service 5: Permission Service
**Purpose**: Grant and manage Lake Formation permissions

**Port**: 8004 | **Replicas**: 3 | **IRSA Role**: PermissionServiceRole

**Features**:
- Table-level access grants
- Column-level access (with masking)
- Row-level filtering
- Tag-based access control

**Example Code**:
```python
lakeformation = boto3.client('lakeformation')

def grant_table_permission(principal_arn, table_name, database_name):
    response = lakeformation.grant_permissions(
        CatalogId='123456789012',
        Principal={
            'DataLakePrincipalIdentifier': principal_arn
        },
        Resource={
            'Table': {
                'CatalogId': '123456789012',
                'DatabaseName': database_name,
                'Name': table_name
            }
        },
        Permissions=['SELECT', 'DESCRIBE'],
        PermissionsWithGrantOption=['SELECT']
    )
    
    # Log event
    eventbridge.put_events(
        Entries=[{
            'Source': 'management-app.permissions',
            'DetailType': 'PermissionGranted',
            'Detail': json.dumps({
                'principal': principal_arn,
                'table': table_name,
                'permissions': ['SELECT', 'DESCRIBE']
            }),
            'EventBusName': 'datamesh-events'
        }]
    )
```

#### 2.6-2.12: Other Services

| Service | Port | Purpose | Replicas |
|---------|------|---------|----------|
| Subscription Service | 8005 | Request/approve data subscriptions | 2 |
| Lake Formation Service | 8006 | Advanced Lake Formation operations | 2 |
| Cross-Account Share Service | 8007 | RAM shares + LF permission sync | 2 |
| Blueprint Deploy Service | 8008 | CloudFormation stack deployment | 3 |
| Data Quality Service | 8009 | Run DQ checks, publish scores | 2 |
| Lineage Tracking Service | 8010 | Ingest + visualize lineage | 2 |
| Iceberg Table Service | 8011 | Time travel, snapshot management | 2 |
| Additional Service | 8012+ | Extended functionality | - |

### ALB Configuration

**Internal Application Load Balancer**:
```
Name: internal-alb
Scheme: internal (no public IP)
Subnets: 10.0.1.0/24, 10.0.2.0/24 (private)

Listeners:
├─ Port 80 (HTTP) → Target Groups
└─ Port 443 (HTTPS) → Target Groups (if needed)

Rules:
├─ /api/domains* → Target Group (domain-svc:8000)
├─ /api/projects* → Target Group (project-svc:8001)
├─ /api/assets* → Target Group (asset-svc:8002)
├─ /api/catalog* → Target Group (catalog-svc:8003)
├─ /api/permissions* → Target Group (permission-svc:8004)
└─ /api/* → Target Group (route to appropriate service)

Health Checks:
├─ Path: /health
├─ Interval: 30 seconds
├─ Healthy Threshold: 2
├─ Unhealthy Threshold: 3
└─ Timeout: 5 seconds
```

### VPC Endpoints

**Purpose**: Private connectivity to AWS services (no internet required)

**Endpoints**:
```
1. com.amazonaws.us-east-1.events (EventBridge)
   └─ Type: Interface (with ENI)
   └─ Private DNS: events.us-east-1.vpce.amazonaws.com

2. com.amazonaws.us-east-1.states (Step Functions)
   └─ Type: Interface (with ENI)

3. com.amazonaws.us-east-1.s3 (S3)
   └─ Type: Gateway (via route table entry)

4. com.amazonaws.us-east-1.dynamodb (DynamoDB)
   └─ Type: Gateway (via route table entry)

5. com.amazonaws.us-east-1.glue (Glue)
   └─ Type: Interface (with ENI)

6. com.amazonaws.us-east-1.datazone (DataZone)
   └─ Type: Interface (with ENI)

7. com.amazonaws.us-east-1.sts (STS - for IRSA token exchange)
   └─ Type: Interface (with ENI)

8. com.amazonaws.us-east-1.kms (KMS)
   └─ Type: Interface (with ENI)
```

### IRSA (IAM Roles for Service Accounts)

**Mechanism**:
```
Pod requests AWS API
    │
    ├─ Kubernetes ServiceAccount has annotation:
    │  eks.amazonaws.com/role-arn: arn:aws:iam::...:role/ServiceRole
    │
    ├─ Pod environment variable:
    │  AWS_ROLE_ARN=arn:aws:iam::...:role/ServiceRole
    │  AWS_WEB_IDENTITY_TOKEN_FILE=/var/run/secrets/eks.amazonaws.com/.../token
    │
    ├─ boto3 (AWS SDK) reads token file
    │
    ├─ Calls STS AssumeRoleWithWebIdentity
    │
    ├─ Returns temporary credentials (access key, secret key, session token)
    │  with 15-minute expiration
    │
    └─ Uses credentials for API calls
```

**OIDC Configuration**:
- EKS cluster has OIDC provider
- ServiceAccount tokens are OIDC-signed JWTs
- IAM trust relationship validates tokens from EKS OIDC provider

---

## Layer 3: Orchestration & Event Layer

### EventBridge Configuration

**Event Bus**: `datamesh-events` (custom event bus)

**Event Sources** (published by EKS pods):
```
1. ProjectCreated
   {
     "source": "management-app.projects",
     "detail-type": "ProjectCreated",
     "detail": {
       "projectId": "proj-abc123",
       "projectName": "Finance Analytics",
       "domainId": "dzd-xyz789",
       "createdBy": "user@company.com",
       "createdAt": "2026-01-14T20:30:00Z"
     }
   }

2. AssetPublished
   {
     "source": "management-app.assets",
     "detail-type": "AssetPublished",
     "detail": {
       "assetId": "asset-def456",
       "assetName": "Transactions Table",
       "s3Location": "s3://data-lake/transactions/",
       "format": "iceberg",
       "glueTable": "finance_db.transactions"
     }
   }

3. SubscriptionRequested
   {
     "source": "management-app.subscriptions",
     "detail-type": "SubscriptionRequested",
     "detail": {
       "requestId": "sub-req-123",
       "assetId": "asset-def456",
       "requestedBy": "user@domain.com",
       "requestedAt": "2026-01-14T20:35:00Z"
     }
   }

4. PermissionChanged
   {
     "source": "management-app.permissions",
     "detail-type": "PermissionGranted|Revoked",
     "detail": {
       "principal": "arn:aws:iam::...:role/AnalystRole",
       "resource": "arn:aws:glue:us-east-1:...:table/finance_db/transactions",
       "permissions": ["SELECT", "DESCRIBE"]
     }
   }
```

### EventBridge Rules

**Rule 1: On Project Created**
```
Name: project-creation-automation
Pattern:
{
  "source": ["management-app.projects"],
  "detail-type": ["ProjectCreated"]
}

Targets:
1. Step Functions: ProjectSetupWorkflow
   └─ Trigger state machine execution
   └─ Pass event detail as input
   └─ Retry policy: 2 attempts, 60 second backoff
   └─ Dead letter queue: SQS for failed executions

2. Lambda: OnProjectCreate
   └─ Create S3 directories
   └─ Setup IAM roles
   └─ Initialize version control
```

**Rule 2: Scheduled - Data Quality Checks**
```
Name: hourly-data-quality-checks
Schedule: rate(1 hour)

Target:
└─ Lambda: TriggerDataQualityChecks
   └─ Runs on every hour
   └─ Invokes EKS data-quality-service
   └─ Reports metrics to CloudWatch
```

**Rule 3: Scheduled - Daily Data Sync**
```
Name: daily-prod-to-dev-sync
Schedule: cron(0 2 * * ? *)  [2 AM UTC daily]

Target:
└─ Lambda: SyncProdToDevData
   └─ Snapshots production data
   └─ Filters (e.g., 10% sample for large tables)
   └─ Replicates to dev/staging environments
```

### Step Functions State Machines

#### State Machine 1: Project Promotion Workflow

```json
{
  "Comment": "Promote project from dev to staging to prod",
  "StartAt": "ValidateSourceEnvironment",
  "States": {
    "ValidateSourceEnvironment": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "http://internal-alb.vpc.local/api/validate",
        "Method": "POST",
        "RequestBody": {
          "projectId.$": "$.projectId",
          "environment": "dev"
        }
      },
      "Next": "DeployToStaging",
      "Catch": [{
        "ErrorEquals": ["States.ALL"],
        "Next": "NotifyFailure"
      }]
    },
    
    "DeployToStaging": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "http://internal-alb.vpc.local/api/deployments",
        "Method": "POST",
        "RequestBody": {
          "projectId.$": "$.projectId",
          "targetEnvironment": "staging",
          "cfnTemplate.$": "$.cfnTemplate"
        }
      },
      "Next": "RunStagingTests",
      "Retry": [{
        "ErrorEquals": ["States.TaskFailed"],
        "IntervalSeconds": 2,
        "MaxAttempts": 3,
        "BackoffRate": 2.0
      }]
    },
    
    "RunStagingTests": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "http://internal-alb.vpc.local/api/tests/run",
        "Method": "POST",
        "RequestBody": {
          "projectId.$": "$.projectId",
          "environment": "staging"
        }
      },
      "Next": "ApprovalGate",
      "TimeoutSeconds": 3600
    },
    
    "ApprovalGate": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken",
      "Parameters": {
        "QueueUrl": "arn:aws:sqs:us-east-1:123456789012:approval-queue",
        "MessageBody": {
          "approvalToken.$": "$$.Task.Token",
          "projectId.$": "$.projectId",
          "message": "Ready to promote to production?"
        }
      },
      "Next": "DeployToProduction"
    },
    
    "DeployToProduction": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "http://internal-alb.vpc.local/api/deployments",
        "Method": "POST",
        "RequestBody": {
          "projectId.$": "$.projectId",
          "targetEnvironment": "prod"
        }
      },
      "Next": "VerifyDeployment"
    },
    
    "VerifyDeployment": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "http://internal-alb.vpc.local/api/verify",
        "Method": "GET",
        "RequestBody": {
          "projectId.$": "$.projectId"
        }
      },
      "Next": "NotifySuccess"
    },
    
    "NotifySuccess": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:promotion-notifications",
        "Subject": "Project Promotion Success",
        "Message.$": "$.projectId"
      },
      "End": true
    },
    
    "NotifyFailure": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:promotion-notifications",
        "Subject": "Project Promotion Failed",
        "Message.$": "$"
      },
      "End": true
    }
  }
}
```

#### State Machine 2: Cross-Account Share Workflow

```
Start
  ↓
CreateRAMShare (AWS RAM resource share)
  ↓
GrantLakeFormationPermissions (sync permissions to accepting account)
  ↓
WaitForAcceptance (poll RAM for acceptance)
  ↓
VerifyAccess (test cross-account access)
  ↓
NotifySubscriber (send SNS notification)
  ↓
End
```

#### State Machine 3: Asset Publication Workflow

```
Start
  ↓
RunDataQualityChecks (validation)
  ↓
ValidateMetadata (required fields)
  ↓
CheckApprovalRequired (based on asset type/sensitivity)
  ├─ If approval needed → WaitForApproval (manual task)
  └─ Else → PublishToCatalog
  ↓
PublishToCatalog (DataZone)
  ↓
IndexInSearch (update search service)
  ↓
NotifySubscribers (email subscribers)
  ↓
End
```

### Lambda Functions (Scheduled)

**Lambda 1: OnProjectCreate**
```python
import boto3
import json
from datetime import datetime

s3 = boto3.client('s3')
iam = boto3.client('iam')
codepipeline = boto3.client('codepipeline')

def lambda_handler(event, context):
    """Triggered when project is created"""
    
    detail = event['detail']
    project_id = detail['projectId']
    domain_id = detail['domainId']
    
    try:
        # 1. Create S3 directories for project
        bucket = f"data-lake-{domain_id}"
        s3.put_object(
            Bucket=bucket,
            Key=f"projects/{project_id}/raw/"
        )
        s3.put_object(
            Bucket=bucket,
            Key=f"projects/{project_id}/processed/"
        )
        s3.put_object(
            Bucket=bucket,
            Key=f"projects/{project_id}/metadata/"
        )
        
        # 2. Create IAM role for project
        trust_policy = {
            "Version": "2012-10-17",
            "Statement": [{
                "Effect": "Allow",
                "Principal": {
                    "AWS": f"arn:aws:iam::123456789012:root"
                },
                "Action": "sts:AssumeRole"
            }]
        }
        
        iam.create_role(
            RoleName=f"project-{project_id}-role",
            AssumeRolePolicyDocument=json.dumps(trust_policy)
        )
        
        # 3. Publish success event
        eventbridge = boto3.client('events')
        eventbridge.put_events(
            Entries=[{
                'Source': 'management-app.lambda',
                'DetailType': 'ProjectSetupCompleted',
                'Detail': json.dumps({
                    'projectId': project_id,
                    'status': 'success'
                })
            }]
        )
        
        return {
            'statusCode': 200,
            'body': json.dumps({'status': 'success'})
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        raise
```

**Lambda 2: TriggerDataQualityChecks**
```python
import boto3
import requests
from datetime import datetime

def lambda_handler(event, context):
    """Triggered hourly to run data quality checks"""
    
    # Call EKS data-quality-service
    alb_endpoint = "http://internal-alb.vpc.local"
    
    try:
        response = requests.post(
            f"{alb_endpoint}/api/data-quality/run",
            json={
                'triggeredBy': 'lambda-scheduler',
                'timestamp': datetime.utcnow().isoformat()
            },
            timeout=300  # 5 minute timeout
        )
        
        result = response.json()
        
        # Publish metrics to CloudWatch
        cloudwatch = boto3.client('cloudwatch')
        cloudwatch.put_metric_data(
            Namespace='ManagementApp',
            MetricData=[{
                'MetricName': 'DataQualityCheckDuration',
                'Value': result.get('duration', 0),
                'Unit': 'Seconds',
                'Timestamp': datetime.utcnow()
            }]
        )
        
        return {
            'statusCode': 200,
            'body': result
        }
        
    except Exception as e:
        print(f"Error invoking EKS service: {str(e)}")
        raise
```

---

## Layer 4: AWS Service Integration

### 4.1 Amazon DataZone

**Purpose**: Self-service data analytics portal and governance

**Integration Points**:
```python
import boto3

datazone = boto3.client('datazone')

# 1. Create domain
response = datazone.create_domain(
    name='finance-domain',
    description='Finance data domain'
)
domain_id = response['id']

# 2. Create project
response = datazone.create_project(
    domainIdentifier=domain_id,
    name='Finance Analytics',
    description='Analyze financial transactions'
)
project_id = response['id']

# 3. Publish asset
response = datazone.create_asset(
    domainIdentifier=domain_id,
    projectIdentifier=project_id,
    name='Transactions Table',
    typeIdentifier='Table',
    typeRevision='1',
    description='Daily financial transactions'
)
asset_id = response['id']

# 4. Publish listing
response = datazone.create_listing(
    domainIdentifier=domain_id,
    projectIdentifier=project_id,
    assetIdentifier=asset_id,
    listingStatus='PUBLISHED'
)

# 5. Create subscription request
response = datazone.create_subscription_request(
    domainIdentifier=domain_id,
    listingIdentifier=listing_id,
    subscriberPrincipal=subscriber_arn
)

# 6. Accept subscription
response = datazone.accept_subscription_request(
    domainIdentifier=domain_id,
    identifier=subscription_id
)
```

### 4.2 AWS Glue

**Purpose**: Data catalog and ETL operations

**Integration Points**:
```python
glue = boto3.client('glue')

# 1. Create database
glue.create_database(
    DatabaseInput={
        'Name': 'finance_db',
        'Description': 'Finance data catalog'
    }
)

# 2. Create table (Iceberg format)
glue.create_table(
    DatabaseName='finance_db',
    TableInput={
        'Name': 'transactions',
        'StorageDescriptor': {
            'Columns': [
                {'Name': 'transaction_id', 'Type': 'bigint'},
                {'Name': 'amount', 'Type': 'decimal(10,2)'},
                {'Name': 'date', 'Type': 'date'}
            ],
            'Location': 's3://data-lake-finance/transactions/',
            'InputFormat': 'org.apache.iceberg.mr.hive.HiveIcebergInputFormat',
            'OutputFormat': 'org.apache.iceberg.mr.hive.HiveIcebergOutputFormat',
            'SerdeInfo': {
                'SerializationLibrary': 'org.apache.iceberg.mr.hive.serde.HiveIcebergSerde'
            }
        },
        'TableType': 'ICEBERG'
    }
)

# 3. Run data quality ruleset evaluation
response = glue.start_data_quality_ruleset_evaluation_run(
    DataSource={
        'GlueTable': {
            'Database': 'finance_db',
            'Table': 'transactions'
        }
    },
    Role='arn:aws:iam::123456789012:role/GlueServiceRole',
    RulesetNames=['FinanceTableRules']
)

# 4. Create crawler (for auto-discovery)
glue.create_crawler(
    Name='s3-to-catalog-crawler',
    Role='arn:aws:iam::123456789012:role/GlueServiceRole',
    DatabaseName='finance_db',
    Targets={
        'S3Targets': [
            {'Path': 's3://data-lake-finance/raw/'}
        ]
    }
)
```

### 4.3 AWS Lake Formation

**Purpose**: Fine-grained access control and permissions

**Integration Points**:
```python
lakeformation = boto3.client('lakeformation')

# 1. Register resource (S3 location)
lakeformation.register_resource(
    ResourceArn='arn:aws:s3:::data-lake-finance/'
)

# 2. Grant table-level permissions
lakeformation.grant_permissions(
    CatalogId='123456789012',
    Principal={
        'DataLakePrincipalIdentifier': 
            'arn:aws:iam::123456789012:role/AnalystRole'
    },
    Resource={
        'Table': {
            'CatalogId': '123456789012',
            'DatabaseName': 'finance_db',
            'Name': 'transactions'
        }
    },
    Permissions=['SELECT', 'DESCRIBE'],
    PermissionsWithGrantOption=['SELECT']
)

# 3. Grant column-level permissions
lakeformation.grant_permissions(
    CatalogId='123456789012',
    Principal={
        'DataLakePrincipalIdentifier': 
            'arn:aws:iam::123456789012:role/AnalystRole'
    },
    Resource={
        'TableWithColumns': {
            'CatalogId': '123456789012',
            'DatabaseName': 'finance_db',
            'Name': 'transactions',
            'ColumnNames': ['transaction_id', 'amount']
        }
    },
    Permissions=['SELECT']
)

# 4. Create data cells filter (PII masking)
lakeformation.create_data_cells_filter(
    TableCellsFilter={
        'DatabaseName': 'finance_db',
        'TableName': 'transactions',
        'Name': 'mask-pii',
        'RowFilter': {
            'FilterExpression': 'user_role = "analyst"'
        },
        'ColumnWildcard': {
            'ExcludedColumnNames': ['customer_ssn', 'credit_card']
        }
    }
)

# 5. Create LF tag
lakeformation.create_lf_tag(
    TagKey='classification',
    TagValues=['public', 'internal', 'confidential']
)

# 6. Assign LF tag to resource
lakeformation.add_lf_tags_to_resource(
    Resource={
        'Table': {
            'CatalogId': '123456789012',
            'DatabaseName': 'finance_db',
            'Name': 'transactions'
        }
    },
    LFTags=[{
        'TagKey': 'classification',
        'TagValues': ['internal']
    }]
)
```

### 4.4 AWS RAM (Resource Access Manager)

**Purpose**: Cross-account resource sharing

**Integration Points**:
```python
ram = boto3.client('ram')

# 1. Create resource share
response = ram.create_resource_share(
    name='finance-data-share',
    resourceArns=[
        'arn:aws:glue:us-east-1:123456789012:catalog',
        'arn:aws:glue:us-east-1:123456789012:database/finance_db'
    ],
    principals=['arn:aws:organizations::123456789012:organization/o-abc123xyz']
)
share_arn = response['resourceShare']['resourceShareArn']

# 2. Accept share (in accepting account)
response = ram.accept_resource_share_invitation(
    resourceShareInvitationArn='arn:aws:ram:us-east-1:...:resource-share-invitation/...'
)

# 3. List associated resources
response = ram.list_resources(
    resourceOwner='OTHER-ACCOUNTS',
    filters=[{
        'name': 'resource-share-arn',
        'values': [share_arn]
    }]
)
```

### 4.5 AWS CloudFormation

**Purpose**: Infrastructure as Code for deployment automation

**Integration Points**:
```python
cf = boto3.client('cloudformation')

# 1. Create stack
response = cf.create_stack(
    StackName='project-prod-stack',
    TemplateBody=cfn_template_json,  # From git repo
    Parameters=[
        {'ParameterKey': 'Environment', 'ParameterValue': 'prod'},
        {'ParameterKey': 'ProjectId', 'ParameterValue': 'proj-123'}
    ],
    Capabilities=['CAPABILITY_NAMED_IAM'],
    Tags=[
        {'Key': 'ManagedBy', 'Value': 'ManagementApp'},
        {'Key': 'ProjectId', 'Value': 'proj-123'}
    ]
)
stack_id = response['StackId']

# 2. Monitor stack creation
waiter = cf.get_waiter('stack_create_complete')
waiter.wait(StackName=stack_id)

# 3. Describe stack
response = cf.describe_stacks(StackName=stack_id)
outputs = {
    output['OutputKey']: output['OutputValue']
    for output in response['Stacks'][0].get('Outputs', [])
}

# 4. Update stack
cf.update_stack(
    StackName=stack_id,
    TemplateBody=updated_template_json
)

# 5. Delete stack
cf.delete_stack(StackName=stack_id)
```

### 4.6 AWS Athena

**Purpose**: SQL queries on Iceberg tables with time travel

**Integration Points**:
```python
athena = boto3.client('athena')

# 1. Run query (Iceberg time travel)
response = athena.start_query_execution(
    QueryString="""
        SELECT * FROM finance_db.transactions
        FOR SYSTEM_TIME AS OF '2026-01-14T10:00:00Z'
        LIMIT 100
    """,
    QueryExecutionContext={'Database': 'finance_db'},
    ResultConfiguration={
        'OutputLocation': 's3://query-results-bucket/queries/'
    },
    ExecutionParameters=[],
    WorkGroup='primary'
)
query_execution_id = response['QueryExecutionId']

# 2. Get results
response = athena.get_query_results(
    QueryExecutionId=query_execution_id
)
```

### 4.7 AWS IAM & KMS

**Purpose**: Identity/access management and encryption key management

**Integration Points**:
```python
iam = boto3.client('iam')
kms = boto3.client('kms')

# 1. Create IAM role for EKS service
iam.create_role(
    RoleName='domain-service-role',
    AssumeRolePolicyDocument=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B716D3041E"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.us-east-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B716D3041E:sub": "system:serviceaccount:management-app:domain-service-sa"
                }
            }
        }]
    })
)

# 2. Attach policies
iam.attach_role_policy(
    RoleName='domain-service-role',
    PolicyArn='arn:aws:iam::aws:policy/CloudWatchLogsFullAccess'
)

# 3. Create KMS key
response = kms.create_key(
    Description='Encryption key for data lake',
    Origin='AWS_KMS',
    MultiRegion=False
)
key_id = response['KeyMetadata']['KeyId']

# 4. Create key alias
kms.create_alias(
    AliasName='alias/data-lake-key',
    TargetKeyId=key_id
)
```

---

## Layer 5: Data Persistence

### DynamoDB Tables

#### Table 1: DeploymentHistory

**Schema**:
```
PK: deploymentId (String)
SK: timestamp (Number)

Attributes:
├─ deploymentId: proj-123-2026-01-14-001
├─ projectId: proj-123
├─ environment: staging | prod
├─ cfnStackId: arn:aws:cloudformation:...
├─ deployedBy: user@company.com
├─ status: IN_PROGRESS | SUCCESS | FAILED
├─ startTime: 1705268400000
├─ endTime: 1705268460000
├─ duration: 60
├─ errorMessage: (if failed)
├─ rollbackReason: (if rolled back)
└─ tags: {ManagedBy: ManagementApp}

Indices:
├─ GSI1: projectId-timestamp-index
│  PK: projectId
│  SK: timestamp
│
└─ GSI2: environment-timestamp-index
   PK: environment
   SK: timestamp

TTL: expirationTime (365 days)
Billing Mode: PAY_PER_REQUEST
```

**Queries**:
```python
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('DeploymentHistory')

# Get deployment
response = table.get_item(
    Key={
        'deploymentId': 'proj-123-2026-01-14-001',
        'timestamp': 1705268400000
    }
)

# Query deployments for project
response = table.query(
    IndexName='projectId-timestamp-index',
    KeyConditionExpression='projectId = :pid AND #ts BETWEEN :ts1 AND :ts2',
    ExpressionAttributeNames={'#ts': 'timestamp'},
    ExpressionAttributeValues={
        ':pid': 'proj-123',
        ':ts1': 1704955200000,  # 30 days ago
        ':ts2': 1705560000000   # today
    }
)

# Put item
table.put_item(
    Item={
        'deploymentId': 'proj-123-2026-01-14-001',
        'timestamp': int(datetime.utcnow().timestamp() * 1000),
        'projectId': 'proj-123',
        'environment': 'prod',
        'status': 'SUCCESS',
        'deployedBy': 'automation@company.com',
        'duration': 60,
        'expirationTime': int((datetime.utcnow().timestamp() + 365*24*3600) * 1000)
    }
)
```

#### Table 2: PromotionRequests

**Schema**:
```
PK: requestId (String)
SK: createdAt (String, ISO-8601)

Attributes:
├─ requestId: prom-req-abc123
├─ projectId: proj-123
├─ sourceEnvironment: dev
├─ targetEnvironment: staging
├─ status: PENDING | APPROVED | REJECTED | COMPLETED
├─ createdBy: user@company.com
├─ approvedBy: reviewer@company.com (nullable)
├─ approvalReason: (nullable)
├─ rejectionReason: (if rejected)
└─ completedAt: (if completed)

Indices:
├─ GSI1: projectId-status-index
│  PK: projectId
│  SK: status
│
└─ GSI2: approverId-createdAt-index
   PK: approvedBy
   SK: createdAt

Billing Mode: PAY_PER_REQUEST
```

#### Table 3: ApprovalWorkflows

**Schema**:
```
PK: workflowId (String)
SK: stepNumber (Number)

Attributes:
├─ workflowId: workflow-xyz789
├─ stepNumber: 1
├─ stepName: validate
├─ status: PENDING | IN_PROGRESS | APPROVED | REJECTED
├─ approver: user@company.com
├─ approvalDeadline: (timestamp)
├─ approvalComment: (optional)
└─ escalationRequired: boolean

Billing Mode: PAY_PER_REQUEST
```

#### Table 4: ConfigurationCache

**Purpose**: Cache frequently accessed configuration

**Schema**:
```
PK: configKey (String)
SK: version (String)

Attributes:
├─ configKey: domain:dzd-xyz789
├─ version: v1
├─ configData: {...} (JSON blob)
├─ expirationTime: (TTL)
└─ lastUpdated: (timestamp)

TTL: expirationTime (1 hour)
Billing Mode: PAY_PER_REQUEST
```

### S3 Buckets

#### Bucket 1: data-lake-{domain}

**Purpose**: Store data lake files (raw, processed, metadata)

**Structure**:
```
s3://data-lake-finance/
├─ raw/
│  ├─ transactions/
│  │  └─ 2026/01/14/data.csv
│  └─ accounts/
│
├─ processed/
│  ├─ transactions/
│  │  └─ (Iceberg snapshots)
│  └─ accounts/
│
├─ metadata/
│  ├─ manifest.json
│  ├─ lineage.json
│  └─ data-catalog.json
│
└─ temp/  (for query results)
   └─ (Athena query outputs)
```

**Versioning**: Enabled
**Encryption**: AES-256 (or KMS)
**Logging**: Access logs to s3://logs-bucket/
**Lifecycle**:
- Transition STANDARD → STANDARD_IA after 30 days
- Transition STANDARD_IA → GLACIER after 90 days
- Delete after 365 days

#### Bucket 2: management-app-artifacts

**Purpose**: Store application configurations, templates, backups

**Structure**:
```
s3://management-app-artifacts/
├─ cloudformation-templates/
│  ├─ project-template.yaml
│  ├─ domain-template.yaml
│  └─ ...
├─ docker-images/
│  ├─ domain-service.tar.gz
│  ├─ project-service.tar.gz
│  └─ ...
├─ kubernetes-manifests/
│  ├─ deployments.yaml
│  ├─ services.yaml
│  └─ ...
└─ backups/
   ├─ dynamodb-backup-2026-01-14.zip
   └─ config-backup-2026-01-14.json
```

**Versioning**: Enabled
**Lifecycle**: Delete incomplete multipart uploads after 7 days

### Glue Data Catalog

**Databases**:
```
1. finance_db
   ├─ transactions (Iceberg)
   ├─ accounts (Iceberg)
   ├─ payments (Iceberg)
   └─ (other tables)

2. marketing_db
   ├─ campaigns (Iceberg)
   ├─ events (Iceberg)
   └─ (other tables)

3. operations_db
   └─ (domain-specific tables)
```

**Table Example**:
```
Table: finance_db.transactions
├─ Format: Iceberg
├─ Location: s3://data-lake-finance/processed/transactions/
├─ Schema:
│  ├─ transaction_id: bigint (primary key)
│  ├─ amount: decimal(10,2)
│  ├─ transaction_date: date
│  ├─ account_id: bigint
│  ├─ merchant_id: string
│  ├─ created_at: timestamp
│  └─ updated_at: timestamp
├─ Partitioning: transaction_date (year, month)
├─ Snapshots: (time travel capability)
│  ├─ 2026-01-14T10:00:00Z
│  ├─ 2026-01-14T09:00:00Z
│  └─ ...
└─ Properties:
   ├─ classification: iceberg
   └─ managed: false (external table)
```

---

## Data Flows & Sequences

### Flow 1: Project Creation

```
1. User Action (via web UI or API)
   POST /api/v1/projects
   {
     "domainId": "dzd-xyz789",
     "projectName": "Finance Analytics",
     "description": "..."
   }

2. API Gateway Routes Request
   ├─ WAF: Validates (allow)
   ├─ Lambda Authorizer: Validates JWT token
   └─ VPC Link: Routes to ALB

3. ALB Routes to Target Pod
   └─ project-svc:8001

4. Project Service (EKS Pod)
   ├─ Creates project in DataZone
   │  datazone.create_project(...)
   │
   ├─ Publishes event to EventBridge
   │  eventbridge.put_events(
   │    source: "management-app.projects",
   │    detail-type: "ProjectCreated",
   │    detail: {...}
   │  )
   │
   └─ Returns response to API Gateway

5. EventBridge Receives Event
   ├─ Matches event pattern
   ├─ Triggers Step Functions state machine
   └─ Triggers Lambda: OnProjectCreate

6. Step Functions Executes Workflow
   ├─ State 1: ValidateSourceEnvironment
   │  └─ Calls EKS service
   │
   ├─ State 2: SetupInfrastructure
   │  └─ Calls CloudFormation
   │
   └─ State 3: NotifyCompletion
      └─ Sends SNS notification

7. Lambda Function Executes
   ├─ Creates S3 directories (s3://data-lake-xxx/projects/proj-123/)
   ├─ Creates IAM role (project-proj-123-role)
   ├─ Initializes git repository
   └─ Publishes success event

8. Response to Client
   ├─ HTTP 200 OK
   └─ Returns projectId
```

### Flow 2: Asset Publication

```
1. User Publishes Asset
   POST /api/v1/assets
   {
     "assetName": "Transactions Table",
     "s3Location": "s3://data-lake/transactions/",
     "glueTable": "finance_db.transactions",
     "format": "iceberg"
   }

2. Asset Service (EKS Pod)
   ├─ Validates metadata (required fields)
   ├─ Runs data quality checks
   │  └─ Calls data-quality-service
   │
   ├─ Validates Glue table exists
   │  glue.get_table(database, table)
   │
   ├─ Publishes to DataZone
   │  datazone.create_asset(...)
   │
   └─ Publishes event: AssetPublished

3. EventBridge Routing
   ├─ Rule: On Asset Published
   └─ Triggers: Step Functions Workflow

4. Asset Publication Workflow
   ├─ RunQualityChecks (if not done)
   ├─ CheckApprovalRequired
   │  ├─ If sensitive data → WaitForApproval
   │  └─ Else → PublishToCatalog
   │
   ├─ PublishToCatalog
   │  datazone.publish_listing(...)
   │
   ├─ IndexForSearch
   │  elasticsearch.index(asset_metadata)
   │
   └─ NotifySubscribers
      └─ SNS email notifications

5. Response to Client
   ├─ HTTP 200 OK
   └─ Returns assetId, publishStatus
```

### Flow 3: Cross-Account Sharing

```
1. User Requests to Share Data
   POST /api/v1/cross-account-share
   {
     "assetId": "asset-def456",
     "targetAccount": "987654321098",
     "permissions": ["SELECT", "DESCRIBE"]
   }

2. Cross-Account Share Service (EKS Pod)
   ├─ Creates RAM resource share
   │  ram.create_resource_share(
   │    resourceArns: [table_arn],
   │    principals: [target_account]
   │  )
   │
   ├─ Grants Lake Formation permissions
   │  lakeformation.grant_permissions(...)
   │
   └─ Publishes event: CrossAccountShareInitiated

3. EventBridge Workflow Triggered
   ├─ Step 1: WaitForRAMAcceptance
   │  └─ Polls RAM API until accepted
   │
   ├─ Step 2: SyncLakeFormationPermissions
   │  └─ Grants perms in accepting account (if cross-account)
   │
   ├─ Step 3: VerifyAccess
   │  └─ Test query in target account
   │
   └─ Step 4: NotifySubscriber
      └─ Send SNS notification

4. Accepting Account (Target)
   ├─ Receives RAM invitation
   ├─ Accepts resource share
   │  ram.accept_resource_share_invitation(...)
   │
   └─ Resources available for querying

5. Response
   ├─ HTTP 200 OK
   └─ Returns shareId, status: PENDING
```

### Flow 4: Data Quality Check (Scheduled)

```
1. EventBridge Scheduled Rule Triggers
   ├─ Schedule: rate(1 hour)
   └─ Target: Lambda (TriggerDataQualityChecks)

2. Lambda Function Executes
   ├─ Calls EKS data-quality-service
   │  POST /api/data-quality/run
   │  {
   │    "datasetsToCheck": ["transactions", "accounts"],
   │    "rules": ["completeness", "uniqueness", "range"]
   │  }
   │
   └─ Waits for response (up to 5 minutes)

3. Data Quality Service (EKS Pod)
   ├─ Loads dataset from S3
   ├─ Executes DQ rules
   │  └─ Glue Data Quality ruleset evaluation
   │
   ├─ Publishes metrics to CloudWatch
   │  cloudwatch.put_metric_data(
   │    MetricName: 'DataQualityScore',
   │    Value: 98.5
   │  )
   │
   ├─ Stores results in DynamoDB
   │  dq_results_table.put_item(...)
   │
   └─ Returns results

4. Lambda Processes Results
   ├─ If DQ score < threshold
   │  └─ Publish event: DataQualityAlert
   │     └─ SNS notification sent to owners
   │
   └─ Updates ConfigurationCache

5. Metrics Visible in CloudWatch
   ├─ Dashboard shows DQ trends
   └─ Alarms trigger if score drops
```

---

## Security Architecture

### 1. Network Security

**Direct Connect**:
- Dedicated fiber connection from on-prem to AWS
- BGP routing (private IP space)
- Redundant connections (2+ for HA)
- No internet exposure

**VPC Design**:
- Private subnets only (no public IPs)
- No internet gateway for general traffic
- VPC endpoints for AWS service access
- Network ACLs restrict traffic

**VPC Endpoints**:
- Type: Interface (ENI-based) for most services
- Type: Gateway for S3, DynamoDB
- Security groups restrict access to EKS pods only
- Private DNS enabled for seamless resolution

**Security Groups**:
```
SG: eks-node-sg
├─ Inbound 443 (from VPC CIDR) - K8s API
├─ Inbound 8000-8012 (from ALB) - Pod ports
├─ Inbound 4194 (from VPC CIDR) - Kubelet
└─ Outbound ALL (to VPC CIDR + VPC endpoints)

SG: alb-sg
├─ Inbound 80 (from VPC Link ENIs)
├─ Inbound 443 (from VPC Link ENIs)
└─ Outbound 8000-8012 (to EKS pods)

SG: vpc-endpoint-sg
├─ Inbound 443 (from EKS pods)
└─ Outbound NONE (stateless return)
```

### 2. Access Control

**API Gateway + Lambda Authorizer**:
- JWT token validation
- User identity extraction
- Group/role-based access
- Fine-grained permission checks

**RBAC (Kubernetes)**:
- ServiceAccount per microservice
- Role bindings limit pod-to-pod access
- Network policies restrict traffic
- Pod security policies enforce best practices

**IAM Roles (IRSA)**:
- ServiceAccount → IAM role trust relationship
- Temporary credentials (15-min session)
- Resource-based policies limit access
- CloudTrail logs all API calls

**Lake Formation**:
- Table-level grants (who can access which tables)
- Column-level grants (access specific columns)
- Row-level filtering (cell-based masking)
- Tag-based access control (LF tags)

### 3. Encryption

**In Transit**:
- TLS 1.2+ for all API calls
- API Gateway enforces HTTPS
- VPC endpoints use TLS tunnels
- Direct Connect encrypted at transport layer

**At Rest**:
- KMS encryption for DynamoDB, S3, Glue Catalog
- Customer-managed keys (CMKS) for sensitive data
- Key rotation enabled (yearly)
- Cross-account key sharing for multi-account setup

**Application Level**:
- Secrets Manager for database passwords
- Environment variables for non-sensitive config
- ConfigMaps for Kubernetes configuration

### 4. Audit & Monitoring

**CloudTrail**:
- Logs all AWS API calls
- Stores logs in S3 (encrypted)
- Enables forensics and compliance

**VPC Flow Logs**:
- Captures all network traffic
- Useful for troubleshooting connectivity
- Stores in CloudWatch Logs or S3

**CloudWatch Logs**:
- Application logs from all services
- Aggregated in single location
- Log Insights for querying
- Metrics and alarms

**EventBridge Audit**:
- All events logged
- Audit trail of data operations
- Tracks who accessed what

---

## Network Architecture

### Direct Connect Setup

```
On-Premises Data Center
    │
    ├─ Customer Router (BGP)
    │
    ├─ AWS Direct Connect Location (e.g., Equinix)
    │  └─ Dedicated fiber (1Gbps, 10Gbps, 40Gbps, 100Gbps)
    │
    ├─ Direct Connect Gateway (DGW)
    │  └─ Abstracts VPC connection details
    │
    ├─ Virtual Private Gateway (VGW)
    │  └─ Sits in VPC
    │
    └─ VPC (AWS)
       └─ Private subnets with EC2/EKS resources
```

**Configuration Example**:
```
Virtual Interface (VIF):
├─ Type: Private VIF (on-prem ↔ VPC)
├─ VLAN: 100
├─ BGP ASN (Customer): 65000
├─ BGP ASN (AWS): 64512
├─ Customer Address: 169.254.10.1/30
├─ AWS Address: 169.254.10.2/30
└─ Prefixes:
   └─ VPC CIDR: 10.0.0.0/16
   └─ On-prem CIDR: 192.168.0.0/16
```

### VPC Architecture

```
┌──────────────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16                                    │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Availability Zone 1 (us-east-1a)                │ │
│  │                                                   │ │
│  │  Private Subnet 1: 10.0.1.0/24                   │ │
│  │  ├─ EKS Worker Node 1                            │ │
│  │  ├─ EKS Pods (domain-svc, project-svc, ...)      │ │
│  │  ├─ VPC Link ENI: 10.0.1.50                      │ │
│  │  ├─ ALB Target: 10.0.1.100                       │ │
│  │  └─ VPC Endpoint ENI: 10.0.1.200                 │ │
│  └─────────────────────────────────────────────────┘ │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Availability Zone 2 (us-east-1b)                │ │
│  │                                                   │ │
│  │  Private Subnet 2: 10.0.2.0/24                   │ │
│  │  ├─ EKS Worker Node 2                            │ │
│  │  ├─ EKS Pods (replicas)                          │ │
│  │  ├─ VPC Link ENI: 10.0.2.50                      │ │
│  │  ├─ ALB Target: 10.0.2.100                       │ │
│  │  └─ VPC Endpoint ENI: 10.0.2.200                 │ │
│  └─────────────────────────────────────────────────┘ │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │
│  │  Route Table (Private Subnets)                    │ │
│  │                                                   │ │
│  │  Destination          Target                      │ │
│  │  ─────────────────────────────────────────────    │ │
│  │  10.0.0.0/16          Local                       │ │
│  │  192.168.0.0/16       vgw-xxx (Direct Connect)    │ │
│  │  0.0.0.0/0            vpce-s3 (S3)                │ │
│  │  169.254.169.254/32   Local (VPC metadata)        │ │
│  └─────────────────────────────────────────────────┘ │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │
│  │  NAT Gateway (Optional, for outbound internet)    │ │
│  │  └─ Only if external API calls needed             │ │
│  └─────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

---

## Infrastructure & Deployment

### Infrastructure as Code (Terraform)

**Directory Structure**:
```
terraform/
├─ main.tf
├─ variables.tf
├─ outputs.tf
├─ vpc.tf
├─ eks.tf
├─ dynamodb.tf
├─ s3.tf
├─ iam.tf
├─ kms.tf
├─ eventbridge.tf
├─ stepfunctions.tf
└─ terraform.tfvars
```

**Example: EKS Cluster**:
```hcl
resource "aws_eks_cluster" "management_cluster" {
  name            = "management-cluster"
  role_arn        = aws_iam_role.eks_cluster_role.arn
  version         = "1.28"

  vpc_config {
    subnet_ids              = aws_subnet.private[*].id
    endpoint_private_access = true
    endpoint_public_access  = false
  }

  enabled_cluster_log_types = ["api", "audit", "authenticator", "controllerManager", "scheduler"]

  tags = {
    ManagedBy = "Terraform"
  }
}

resource "aws_eks_node_group" "workers" {
  cluster_name    = aws_eks_cluster.management_cluster.name
  node_group_name = "worker-nodes"
  node_role_arn   = aws_iam_role.eks_node_role.arn
  subnet_ids      = aws_subnet.private[*].id

  scaling_config {
    desired_size = 3
    max_size     = 10
    min_size     = 3
  }

  instance_types = ["t3.xlarge"]

  tags = {
    ManagedBy = "Terraform"
  }
}
```

### Kubernetes Manifests (Helm)

**Helm Chart Structure**:
```
helm/management-app/
├─ Chart.yaml
├─ values.yaml
├─ values-prod.yaml
├─ templates/
│  ├─ namespace.yaml
│  ├─ serviceaccount.yaml
│  ├─ configmap.yaml
│  ├─ secret.yaml
│  ├─ deployment.yaml
│  ├─ service.yaml
│  ├─ ingress.yaml
│  ├─ hpa.yaml
│  └─ pdb.yaml (Pod Disruption Budget)
└─ subcharts/
   ├─ domain-service/
   ├─ project-service/
   └─ ...
```

**Deployment Process**:
```
1. Developer commits code to GitLab
   └─ Triggers CI/CD pipeline

2. CI Pipeline
   ├─ Build Docker image
   ├─ Push to ECR
   ├─ Run tests
   ├─ Security scan (trivy)
   └─ Tag as :version

3. CD Pipeline (ArgoCD)
   ├─ Fetch Helm chart
   ├─ Render manifests
   ├─ Apply to EKS cluster
   ├─ Monitor rollout
   └─ Rollback on failure

4. Kubernetes Applies Manifests
   ├─ Creates Deployment pods
   ├─ Health checks (liveness/readiness probes)
   ├─ Updates Service endpoints
   └─ Routes traffic to new pods

5. Monitoring
   ├─ Prometheus scrapes metrics
   ├─ Alerts fire if unhealthy
   └─ Logs streamed to CloudWatch
```

---

## Monitoring & Operations

### CloudWatch Dashboards

**Main Dashboard**:
```
┌─ EKS Cluster Health
│  ├─ Node Count (desired vs actual)
│  ├─ CPU Usage (%)
│  ├─ Memory Usage (%)
│  └─ Network I/O (bytes/sec)
│
├─ Application Metrics
│  ├─ API Request Rate (req/sec)
│  ├─ API Latency (p50, p99, max)
│  ├─ Error Rate (%)
│  └─ Pod Restart Count
│
├─ Data Platform Metrics
│  ├─ EventBridge Events (events/sec)
│  ├─ Step Functions Executions (running, succeeded, failed)
│  ├─ Lambda Invocations (count, duration, errors)
│  └─ DynamoDB Read/Write Units
│
├─ Data Lake Health
│  ├─ S3 Objects (count, size)
│  ├─ Glue Jobs (running, succeeded, failed)
│  ├─ Athena Queries (running, succeeded, failed)
│  └─ Data Quality Scores (%)
│
└─ System Health
   ├─ Uptime (%)
   ├─ Disk Usage (%)
   ├─ Network Connectivity
   └─ Direct Connect Status
```

### Alarms

**Critical Alarms**:
```
1. EKS Node Status
   └─ Alert if nodes unavailable or NotReady

2. Pod Restart Count
   └─ Alert if > 5 restarts in 1 hour

3. API Error Rate
   └─ Alert if > 5% for 5 minutes

4. Data Quality Score
   └─ Alert if < 90% for 2 consecutive checks

5. DynamoDB Throttling
   └─ Alert if ConsumedWriteCapacity > provisioned

6. Lambda Duration
   └─ Alert if > 80% of timeout threshold

7. Direct Connect Status
   └─ Alert if connection DOWN
```

### Logging

**Log Aggregation**:
```
CloudWatch Logs
├─ /aws/eks/management-cluster/cluster
│  └─ Kubernetes API server logs
│
├─ /aws/eks/management-cluster/audit
│  └─ API audit trails
│
├─ /aws/eks/management-cluster/pods
│  ├─ /management-app/domain-service
│  ├─ /management-app/project-service
│  └─ ...
│
├─ /aws/lambda/ManagementApp
│  ├─ /OnProjectCreate
│  ├─ /TriggerDataQualityChecks
│  └─ ...
│
├─ /aws/states/datamesh-workflows
│  ├─ /PromotionWorkflow
│  ├─ /CrossAccountShareWorkflow
│  └─ ...
│
└─ /aws/apigateway/datamesh-api
   └─ API access logs
```

**Log Insights Queries**:
```
# High latency endpoints
fields @timestamp, @duration
| filter @duration > 1000
| stats avg(@duration) by @endpoint

# Error analysis
fields @timestamp, @message, @stack
| filter @message like /ERROR|Exception/
| stats count() by @message

# API Gateway 4xx/5xx errors
fields @timestamp, @httpMethod, @status
| filter @status >= 400
| stats count() by @status
```

---

## Scalability & Performance

### Horizontal Scaling

**EKS Cluster**:
- **Min nodes**: 3
- **Max nodes**: 10
- **Scaling trigger**: CPU > 70% or Memory > 80%
- **Scale-down**: After 10 minutes of <30% utilization

**Microservices (HPA)**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: project-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: project-service
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**DynamoDB**:
- **Billing Mode**: PAY_PER_REQUEST (auto-scales)
- **Max capacity**: AWS account limits (can request increase)

**Lambda**:
- **Concurrent execution limit**: 1000 (per account, per region)
- **Reserved concurrency**: Set for critical functions
- **Provisioned concurrency**: For predictable workloads

### Performance Optimization

**Caching**:
- ConfigurationCache (DynamoDB with TTL)
- API response caching (5 min)
- Kubernetes DNS caching

**Connection Pooling**:
- boto3 session reuse
- HTTP connection pools
- Database connection pooling

**Batch Processing**:
- DynamoDB batch writes (BatchWriteItem)
- Glue batch operations
- Lambda reserved concurrency

### Rate Limiting

**API Gateway**:
- 10,000 requests/sec (default)
- Custom rate limiting via Lambda Authorizer
- Throttle policy per user/API key

**Service Level**:
- Queue-based retry (SQS for failed requests)
- Circuit breaker pattern (fail fast)
- Exponential backoff for retries

---

## Disaster Recovery

### RTO/RPO Targets

| Component | RTO | RPO |
|-----------|-----|-----|
| EKS Cluster | 2 hours | 0 (multi-AZ) |
| DynamoDB | < 1 minute | < 1 minute (global tables) |
| S3 Data Lake | < 1 hour | < 1 hour (versioning) |
| Glue Catalog | 2 hours | 0 (AWS managed) |
| Configuration | 30 minutes | 0 (version controlled) |

### Backup Strategy

**DynamoDB**:
```
Point-in-time Recovery (PITR)
├─ Enabled for all tables
├─ Retention: 35 days
└─ Restore to any point in time

On-demand Backups
├─ Daily backup (via Lambda)
├─ Retention: 30 days
└─ Can restore tables
```

**S3**:
```
Versioning
├─ Enabled on all buckets
├─ Allows recovery of deleted objects
└─ Lifecycle: 30-day retention

Cross-Region Replication
├─ Replicate to secondary region
├─ async updates
└─ Backup in case of region failure
```

**Application Code**:
```
Git Repository (GitLab)
├─ All code version controlled
├─ Protected main branch
├─ Tag releases
└─ Can rollback to any version

Docker Images
├─ Tagged with version
├─ Stored in ECR
├─ Multi-region replication possible
└─ Can redeploy any version
```

### Failover Procedure

**1. EKS Cluster Failure**:
```
1. Detect failure (unhealthy nodes, pod restarts)
2. Auto-scaling groups recreate nodes
3. Kubernetes reschedulesods
4. Load balancer routes to new pods
5. DataSync restores state from DynamoDB
```

**2. Region Failure**:
```
1. Manual failover to secondary region
2. Restore DynamoDB from backup
3. Redeploy EKS cluster (Terraform)
4. Update DNS to point to new region
5. Resume operations
```

---

## Cost Estimation

### Monthly Costs (Rough Estimates)

| Component | Usage | Cost |
|-----------|-------|------|
| **EKS Cluster** | 3-10 t3.xlarge | $300-1000 |
| **EKS Data Transfer** | 10 TB/month | $500-1000 |
| **DynamoDB** | 10M requests | $500-1000 |
| **S3 Storage** | 100 TB | $2300 |
| **S3 Requests** | 100M requests | $500 |
| **Lambda** | 10M invocations | $200 |
| **Step Functions** | 1M transitions | $50 |
| **EventBridge** | 100M events | $50 |
| **VPC Endpoints** | 6 endpoints | $400 |
| **CloudWatch** | Logs + metrics | $500 |
| **Direct Connect** | 1 10Gbps port | $3000-5000 |
| **KMS** | Key + requests | $100 |
| **DataZone** | 10 domains | $500 |
| **Glue** | Catalog + jobs | $200 |
| **Athena** | Queries | $100 |
| **Total (Estimated)** | | **$10,000-15,000/month** |

### Cost Optimization

- **Reserved Capacity**: EKS nodes (1-year commitments)
- **Spot Instances**: Non-critical workloads (70% discount)
- **Compute Savings Plans**: Lambda and general compute
- **Data Transfer**: Consolidate to single region
- **Log Retention**: Archive old logs to Glacier

---

## Conclusion

This architecture provides a **scalable, secure, and highly available** Data Mesh management platform on AWS. Key strengths:

1. ✅ **Microservices-based**: Independent scaling, deployment
2. ✅ **Event-driven**: Async workflows, loose coupling
3. ✅ **Cloud-native**: Leverages AWS managed services
4. ✅ **Secure**: Private networking, encryption, RBAC
5. ✅ **Observable**: Comprehensive logging and monitoring
6. ✅ **Cost-effective**: Pay-as-you-go pricing model

Next steps:
1. Finalize Terraform code
2. Set up CI/CD pipelines
3. Configure monitoring and alerts
4. Conduct security review
5. Plan migration from legacy system
6. Train operations team

