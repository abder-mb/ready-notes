# Data Mesh Implementation - SageMaker Unified Studio HLD Architecture

## 1. MULTI-ACCOUNT ENVIRONMENT STRUCTURE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     AWS ORGANIZATION (Root Account)                          │
│                                                                              │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐     │
│  │  DEV ACCOUNT     │    │ STAGING ACCOUNT  │    │  PROD ACCOUNT    │     │
│  │  123456789012    │    │  234567890123    │    │  345678901234    │     │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘     │
│           │                       │                       │                 │
│           ▼                       ▼                       ▼                 │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐     │
│  │ SMUS Domain      │    │ SMUS Domain      │    │ SMUS Domain      │     │
│  │ acme-dev-        │    │ acme-staging-    │    │ acme-prod-       │     │
│  │ unified-domain   │    │ unified-domain   │    │ unified-domain   │     │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. DOMAIN STRUCTURE (Per Environment)

```
┌──────────────────────────────────────────────────────────────────────┐
│  SMUS DOMAIN: acme-{env}-unified-domain                              │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  ROOT DOMAIN UNIT                                              │ │
│  │  - Top level organizational boundary                           │ │
│  │  - Domain administrators                                       │ │
│  │                                                                 │ │
│  │  ├─► BUSINESS UNIT DOMAIN UNITS                                │ │
│  │  │                                                              │ │
│  │  │   ┌─────────────────────┐  ┌─────────────────────┐         │ │
│  │  │   │ finance-domain-unit │  │ sales-domain-unit   │         │ │
│  │  │   │                     │  │                     │         │ │
│  │  │   │  ├─► Projects:      │  │  ├─► Projects:      │         │ │
│  │  │   │  │   - finance-     │  │  │   - sales-       │         │ │
│  │  │   │  │     customer360  │  │  │     forecasting  │         │ │
│  │  │   │  │   - finance-     │  │  │   - sales-       │         │ │
│  │  │   │  │     reporting    │  │  │     analytics    │         │ │
│  │  │   │  │                  │  │  │                  │         │ │
│  │  │   │  └─► Team DUs:      │  │  └─► Team DUs:      │         │ │
│  │  │   │      - finance-     │  │      - sales-       │         │ │
│  │  │   │        analytics-DU │  │        ops-DU       │         │ │
│  │  │   └─────────────────────┘  └─────────────────────┘         │ │
│  │  │                                                              │ │
│  │  └─► ┌─────────────────────┐  ┌─────────────────────┐         │ │
│  │      │ ops-domain-unit     │  │ data-platform-DU    │         │ │
│  │      └─────────────────────┘  └─────────────────────┘         │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

## 3. CATALOG & GLOSSARY ORGANIZATION

```
┌────────────────────────────────────────────────────────────────────────┐
│                       CATALOG LAYER                                     │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  CENTRAL CATALOG: acme-{env}-central-catalog                     │ │
│  │  (Business Catalog - Domain-wide asset discovery)               │ │
│  │                                                                   │ │
│  │  • Published Assets from all domains                             │ │
│  │  • Searchable by all authorized users                            │ │
│  │  • Contains metadata, lineage, quality scores                    │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                   │                                    │
│                                   ▼                                    │
│  ┌──────────────────────┐  ┌──────────────────────┐                  │
│  │ DOMAIN CATALOGS      │  │ DOMAIN CATALOGS      │                  │
│  │ finance-{env}-       │  │ sales-{env}-         │  ┌────────────┐  │
│  │ catalog              │  │ catalog              │  │ ops-{env}- │  │
│  │                      │  │                      │  │ catalog    │  │
│  │ - GL Transactions    │  │ - CRM Data           │  │ - Inventory│  │
│  │ - P&L Reports        │  │ - Opportunities      │  │ - Supply   │  │
│  │ - Budget Data        │  │ - Quotes             │  │ - Orders   │  │
│  └──────────────────────┘  └──────────────────────┘  └────────────┘  │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│                       GLOSSARY LAYER                                    │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  ENTERPRISE GLOSSARY: acme-enterprise-glossary                   │ │
│  │                                                                   │ │
│  │  Organization-wide terms:                                         │ │
│  │  • Customer, Product, Revenue, Transaction, Account, etc.        │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                   │                                    │
│                                   ▼                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │ finance-glossary │  │ sales-glossary   │  │ ops-glossary     │   │
│  │                  │  │                  │  │                  │   │
│  │ - EBITDA         │  │ - Lead           │  │ - SKU            │   │
│  │ - COGS           │  │ - Opportunity    │  │ - Fulfillment    │   │
│  │ - ARR            │  │ - Pipeline       │  │ - Warehouse      │   │
│  │ - CAPEX          │  │ - Quota          │  │ - Shipment       │   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
└────────────────────────────────────────────────────────────────────────┘
```

## 4. GIT & CI/CD PIPELINE

```
┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│  ON-PREMISES     │         │  AWS CODE        │         │  CI/CD PIPELINE  │
│  GIT             │────────►│  CONNECTIONS     │────────►│                  │
│                  │ Webhook │                  │ Trigger │  CodePipeline    │
│  GitLab Self-    │         │  Connection:     │         │                  │
│  Managed         │         │  - GitLab SM     │         │  ┌────────────┐  │
│    OR            │         │  - OAuth/PAT     │         │  │  Source    │  │
│  GitHub          │         │  - Secure tunnel │         │  └─────┬──────┘  │
│  Enterprise      │         │                  │         │        ▼         │
│                  │         │                  │         │  ┌────────────┐  │
│  git.company.    │         │                  │         │  │  Build     │  │
│  internal        │         │                  │         │  │  CodeBuild │  │
└──────────────────┘         └──────────────────┘         │  └─────┬──────┘  │
                                                           │        ▼         │
                                                           │  ┌────────────┐  │
                                                           │  │ Deploy DEV │  │
                                                           │  │ (Auto)     │  │
                                                           │  └─────┬──────┘  │
                                                           │        ▼         │
                                                           │  ┌────────────┐  │
                                                           │  │Deploy STG  │  │
                                                           │  │(Approval)  │  │
                                                           │  └─────┬──────┘  │
                                                           │        ▼         │
                                                           │  ┌────────────┐  │
                                                           │  │Deploy PROD │  │
                                                           │  │(Approval)  │  │
                                                           │  └────────────┘  │
                                                           └──────────────────┘
                                                                    │
                                        ┌───────────────────────────┼───────────────────────────┐
                                        ▼                           ▼                           ▼
                              ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
                              │  DEV Account     │      │ STAGING Account  │      │  PROD Account    │
                              │  Deploy via      │      │ Deploy via       │      │  Deploy via      │
                              │  CloudFormation  │      │ CloudFormation   │      │  CloudFormation  │
                              └──────────────────┘      └──────────────────┘      └──────────────────┘
```

## 5. MANAGEMENT APPLICATION ARCHITECTURE

```
┌────────────────────────────────────────────────────────────────────────────┐
│                   MANAGEMENT APPLICATION                                    │
│                   (Python + boto3.datazone SDK)                            │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  APPLICATION LAYER                                                    │ │
│  │                                                                        │ │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐           │ │
│  │  │ REST API     │    │ Web UI       │    │ CLI Tool     │           │ │
│  │  │ (FastAPI)    │    │ (Flask/React)│    │ (Typer)      │           │ │
│  │  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘           │ │
│  │         │                    │                    │                   │ │
│  │         └────────────────────┼────────────────────┘                   │ │
│  │                              ▼                                        │ │
│  │         ┌────────────────────────────────────────────┐               │ │
│  │         │    BUSINESS LOGIC LAYER                    │               │ │
│  │         │                                             │               │ │
│  │         │  • Asset Management Service                │               │ │
│  │         │  • Project Management Service              │               │ │
│  │         │  • Subscription Management Service         │               │ │
│  │         │  • Permission Management Service           │               │ │
│  │         │  • Catalog Management Service              │               │ │
│  │         └──────────────────┬─────────────────────────┘               │ │
│  │                            ▼                                          │ │
│  │         ┌────────────────────────────────────────────┐               │ │
│  │         │    boto3.client('datazone')                │               │ │
│  │         └──────────────────┬─────────────────────────┘               │ │
│  └────────────────────────────┼──────────────────────────────────────────┘ │
│                               ▼                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │  BOTO3 DATAZONE SDK OPERATIONS                                      │  │
│  │                                                                      │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │  │
│  │  │ ASSET OPS       │  │ PROJECT OPS     │  │ SUBSCRIPTION    │    │  │
│  │  │                 │  │                 │  │ OPS             │    │  │
│  │  │ create_asset    │  │ create_project  │  │ create_sub_req  │    │  │
│  │  │ update_asset    │  │ update_project  │  │ accept_sub      │    │  │
│  │  │ delete_asset    │  │ delete_project  │  │ reject_sub      │    │  │
│  │  │ publish_listing │  │ add_member      │  │ revoke_sub      │    │  │
│  │  │ search_assets   │  │ remove_member   │  │ grant_sub       │    │  │
│  │  │ attach_metadata │  │ list_projects   │  │ list_subs       │    │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘    │  │
│  │                                                                      │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │  │
│  │  │ CATALOG OPS     │  │ PERMISSION OPS  │  │ PUBLICATION     │    │  │
│  │  │                 │  │                 │  │ OPS             │    │  │
│  │  │ create_catalog  │  │ grant_access    │  │ publish_asset   │    │  │
│  │  │ update_catalog  │  │ revoke_access   │  │ unpublish_asset │    │  │
│  │  │ list_catalogs   │  │ list_permissions│  │ update_listing  │    │  │
│  │  │ search_catalog  │  │ update_policy   │  │ list_published  │    │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘    │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                        ┌────────────────────────┐
                        │  AWS DataZone API      │
                        │  (SMUS Backend)        │
                        └────────────────────────┘
```

## 6. AWS SERVICES STACK

```
┌────────────────────────────────────────────────────────────────────────┐
│                    CORE AWS SERVICES                                    │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  DATA PLATFORM SERVICES                                          │ │
│  │                                                                   │ │
│  │  • Amazon DataZone / SageMaker Unified Studio (Core)             │ │
│  │  • AWS Lake Formation (Data Governance & Access Control)         │ │
│  │  • AWS Glue (Data Catalog, ETL, Crawler)                         │ │
│  │  • Amazon Athena (SQL Analytics)                                 │ │
│  │  • Amazon Redshift (Data Warehouse)                              │ │
│  │  • Amazon S3 (Data Lake Storage)                                 │ │
│  │  • Amazon EMR (Big Data Processing)                              │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  AUTOMATION & DEVOPS SERVICES                                    │ │
│  │                                                                   │ │
│  │  • AWS CloudFormation (Infrastructure as Code)                   │ │
│  │  • AWS CodePipeline (CI/CD Orchestration)                        │ │
│  │  • AWS CodeBuild (Build & Test)                                  │ │
│  │  • AWS CodeDeploy (Deployment)                                   │ │
│  │  • AWS CodeConnections (Git Integration)                         │ │
│  │  • AWS Lambda (Serverless Functions)                             │ │
│  │  • Amazon EventBridge (Event-Driven Automation)                  │ │
│  │  • AWS Step Functions (Workflow Orchestration)                   │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  GOVERNANCE & SECURITY SERVICES                                  │ │
│  │                                                                   │ │
│  │  • AWS IAM (Identity & Access Management)                        │ │
│  │  • AWS Organizations (Multi-Account Management)                  │ │
│  │  • AWS KMS (Key Management Service - Encryption)                 │ │
│  │  • AWS Secrets Manager (Credentials Management)                  │ │
│  │  • AWS CloudTrail (Audit Logging)                                │ │
│  │  • AWS Config (Compliance Monitoring)                            │ │
│  │  • AWS Security Hub (Security Posture)                           │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │  NETWORKING SERVICES                                             │ │
│  │                                                                   │ │
│  │  • Amazon VPC (Virtual Private Cloud)                            │ │
│  │  • AWS PrivateLink (Private Connectivity)                        │ │
│  │  • AWS VPN / Direct Connect (On-Prem Connectivity)               │ │
│  └──────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

## 7. PROJECT STRUCTURE & NAMING

```
PROJECT NAMING PATTERN: <domain-unit>-<use-case>-<env>-project

Examples:
┌─────────────────────────────────────────────────────────────┐
│ Domain Unit          Use Case        Env      Project Name  │
├─────────────────────────────────────────────────────────────┤
│ finance          →   customer360  →  dev  →  finance-       │
│                                               customer360-   │
│                                               dev-project    │
│                                                              │
│ sales            →   forecasting  →  prod →  sales-         │
│                                               forecasting-   │
│                                               prod-project   │
│                                                              │
│ ops              →   inventory    →  stg  →  ops-inventory- │
│                                               stg-project    │
└─────────────────────────────────────────────────────────────┘

PROJECT STRUCTURE:
┌──────────────────────────────────────────────────────────────────┐
│  PROJECT: finance-customer360-dev-project                        │
│                                                                   │
│  ├─► Project Profile                                             │
│  │   - All capabilities OR SQL analytics OR Custom              │
│  │                                                               │
│  ├─► Project Members                                             │
│  │   - Owners: finance-team-leads group                         │
│  │   - Contributors: finance-analysts group                     │
│  │   - Viewers: executives group                                │
│  │                                                               │
│  ├─► Blueprints (Enabled)                                        │
│  │   - Tooling (JupyterLab, Code Editor)                        │
│  │   - LakehouseDatabase (AWS Glue)                             │
│  │   - RedshiftServerless                                        │
│  │   - LakehouseCatalog                                          │
│  │   - EMRServerless                                             │
│  │                                                               │
│  ├─► Assets (Inventory)                                          │
│  │   - Customer Master Data (private)                           │
│  │   - Transaction History (private)                            │
│  │   - Customer 360 View (published to catalog)                 │
│  │                                                               │
│  ├─► Subscriptions                                               │
│  │   - Subscribed to: sales-opportunity-data                    │
│  │   - Subscribed to: product-catalog                           │
│  │                                                               │
│  └─► Git Repository                                              │
│      - notebooks/                                                │
│      - queries/                                                  │
│      - workflows/                                                │
│      - transformations/                                          │
└──────────────────────────────────────────────────────────────────┘
```

## 8. DATA FLOW & SUBSCRIPTION MODEL

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA MESH DATA FLOW                               │
│                                                                      │
│  PRODUCER PROJECT                  CENTRAL CATALOG                  │
│  (Data Product Owner)                                               │
│                                                                      │
│  ┌──────────────────┐                                               │
│  │ finance-         │                                               │
│  │ reporting-       │   1. Publish                                  │
│  │ prod-project     │      Asset                                    │
│  │                  │ ───────────►  ┌─────────────────────┐        │
│  │ Asset:           │               │ Central Catalog     │        │
│  │ - GL_Ledger_Data │               │                     │        │
│  │ - Metadata       │               │ Published Assets:   │        │
│  │ - Quality: 95%   │               │ • GL_Ledger_Data    │        │
│  │ - Owner: Finance │               │ • Customer_Master   │        │
│  └──────────────────┘               │ • Product_Catalog   │        │
│                                      │ • Sales_Trans       │        │
│         │                            └─────────────────────┘        │
│         │                                      │                    │
│         │ 4. Grant                             │ 2. Search &        │
│         │    Access                            │    Subscribe       │
│         │                                      │                    │
│         ▼                                      ▼                    │
│  ┌──────────────────┐               ┌─────────────────────┐        │
│  │ Lake Formation   │               │ CONSUMER PROJECT    │        │
│  │ Permissions      │               │                     │        │
│  │                  │◄──────────────│ sales-analytics-    │        │
│  │ Grant:           │ 3. Request    │ prod-project        │        │
│  │ sales-analytics  │    Access     │                     │        │
│  │ → GL_Ledger_Data │               │ Subscribed to:      │        │
│  │   (READ)         │               │ • GL_Ledger_Data    │        │
│  └──────────────────┘               └─────────────────────┘        │
│                                                                      │
│  SUBSCRIPTION WORKFLOW:                                             │
│  1. Consumer searches catalog                                       │
│  2. Consumer submits subscription request                           │
│  3. Producer approves/rejects request                               │
│  4. System grants access via Lake Formation                         │
│  5. Consumer can query/analyze the data                             │
└─────────────────────────────────────────────────────────────────────┘
```

## 9. COMPLETE NAMING CONVENTION SUMMARY

```
┌────────────────────────────────────────────────────────────────────┐
│                  NAMING CONVENTIONS REFERENCE                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  DOMAINS:                                                           │
│    Pattern: <org>-<env>-unified-domain                             │
│    Examples: acme-dev-unified-domain                               │
│              acme-staging-unified-domain                           │
│              acme-prod-unified-domain                              │
│                                                                     │
│  DOMAIN UNITS:                                                      │
│    Root: Root Domain Unit (system default)                         │
│    Business: <business-unit>-domain-unit                           │
│    Examples: finance-domain-unit, sales-domain-unit                │
│    Teams: <bu>-<team>-domain-unit                                  │
│    Examples: finance-analytics-domain-unit                         │
│                                                                     │
│  CATALOGS:                                                          │
│    Central: <org>-<env>-central-catalog                            │
│    Examples: acme-prod-central-catalog                             │
│    Domain: <domain>-<env>-catalog                                  │
│    Examples: finance-prod-catalog, sales-dev-catalog               │
│                                                                     │
│  GLOSSARIES:                                                        │
│    Enterprise: <org>-enterprise-glossary                           │
│    Examples: acme-enterprise-glossary                              │
│    Domain: <domain>-glossary                                       │
│    Examples: finance-glossary, sales-glossary, ops-glossary        │
│                                                                     │
│  PROJECTS:                                                          │
│    Pattern: <domain-unit>-<use-case>-<env>-project                │
│    Examples: finance-customer360-dev-project                       │
│              sales-forecasting-prod-project                        │
│              ops-inventory-staging-project                         │
│              data-platform-etl-dev-project                         │
│                                                                     │
│  ASSETS:                                                            │
│    Pattern: <domain>_<dataset>_<type>                             │
│    Examples: finance_gl_transactions_table                         │
│              sales_opportunities_view                              │
│              customer_master_dataset                               │
│                                                                     │
│  PROJECT PROFILES:                                                  │
│    Pattern: <capability>-<env>-profile                            │
│    Examples: all-capabilities-dev-profile                          │
│              sql-analytics-prod-profile                            │
│              genai-dev-profile                                     │
└────────────────────────────────────────────────────────────────────┘
```

## 10. SECURITY & ACCESS CONTROL

```
┌────────────────────────────────────────────────────────────────────┐
│                    IAM & SECURITY ARCHITECTURE                      │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  AWS ORGANIZATIONS                                            │ │
│  │                                                                │ │
│  │  ┌─────────────────────────────────────────────────────────┐ │ │
│  │  │  Management Account (Root)                              │ │ │
│  │  │  - Organization SCPs (Service Control Policies)         │ │ │
│  │  │  - Cross-account IAM roles                              │ │ │
│  │  └─────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │ │
│  │  │ Dev OU       │  │ Staging OU   │  │ Prod OU      │       │ │
│  │  │              │  │              │  │              │       │ │
│  │  │ - Dev Acct   │  │ - Stg Acct   │  │ - Prod Acct  │       │ │
│  │  │ - SCPs       │  │ - SCPs       │  │ - SCPs       │       │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘       │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  IAM IDENTITY CENTER (SSO)                                    │ │
│  │                                                                │ │
│  │  Users/Groups           Permission Sets      SMUS Domains     │ │
│  │  ┌────────────────┐                         ┌──────────────┐ │ │
│  │  │ finance-team   │ ──► DataAnalyst     ──► │ Finance      │ │ │
│  │  │ sales-team     │     PermissionSet       │ Projects     │ │ │
│  │  │ data-engineers │ ──► DataEngineer    ──► │ Platform     │ │ │
│  │  │ executives     │     PermissionSet       │ Projects     │ │ │
│  │  └────────────────
