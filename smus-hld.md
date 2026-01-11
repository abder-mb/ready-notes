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








# Explication Détaillée : Architecture Multi-Compte AWS

## Vue d'Ensemble

Cette section représente l'architecture **multi-compte** qui est la **fondation** de votre implémentation Data Mesh. C'est le niveau le plus haut de votre infrastructure.

---

## Pourquoi 3 Comptes AWS Séparés ?

### 1. **Isolation & Sécurité**
- Chaque environnement (Dev, Staging, Prod) est **complètement isolé**
- Si un problème survient en Dev → **aucun impact** sur Prod
- Limites de sécurité strictes entre environnements
- En cas de compromission d'un compte → les autres restent protégés

### 2. **Contrôle des Coûts**
- Facturation séparée par environnement
- Vous pouvez voir exactement combien coûte Dev vs Prod
- Budgets et alertes par compte
- Exemple : Budget Dev = 5K€/mois, Prod = 50K€/mois

### 3. **Gestion des Accès**
- **Dev** : Accès large pour les développeurs (ils peuvent expérimenter)
- **Staging** : Accès limité (pour tests de validation)
- **Prod** : Accès très restreint (seulement les ops et déploiements automatisés)

---

## Structure Détaillée

```
┌────────────────────────────────────────────────────────────────┐
│  AWS ORGANIZATION (Compte Root/Management)                     │
│  - C'est le "parent" qui gère les 3 comptes enfants           │
│  - Politiques centralisées (SCPs - Service Control Policies)   │
│  - Facturation consolidée                                      │
│  - Gestion centralisée des utilisateurs (IAM Identity Center)  │
└────────────────────────────────────────────────────────────────┘
         │
         ├──────────────────┬──────────────────┬──────────────────
         │                  │                  │
         ▼                  ▼                  ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ DEV ACCOUNT     │  │ STAGING ACCOUNT │  │ PROD ACCOUNT    │
│ 123456789012    │  │ 234567890123    │  │ 345678901234    │
│                 │  │                 │  │                 │
│ Purpose:        │  │ Purpose:        │  │ Purpose:        │
│ - Développement │  │ - Tests         │  │ - Production    │
│ - Expérimentation│ │ - Validation    │  │ - Clients réels │
│ - Apprentissage │  │ - UAT           │  │ - Données réelles│
│                 │  │                 │  │                 │
│ Accès:          │  │ Accès:          │  │ Accès:          │
│ - Développeurs  │  │ - QA Team       │  │ - Ops seulement │
│ - Data Engineers│  │ - Product Owner │  │ - CI/CD auto    │
│ - Large accès   │  │ - Accès modéré  │  │ - Très restreint│
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

## Chaque Compte Contient un Domaine SMUS

```
COMPTE DEV
└─► Domaine SMUS: acme-dev-unified-domain
    │
    ├─► Projets Dev
    │   - finance-customer360-dev-project
    │   - sales-forecasting-dev-project
    │   - data-platform-etl-dev-project
    │
    ├─► Données Dev
    │   - Données de test
    │   - Données synthétiques
    │   - Échantillons de prod (anonymisés)
    │
    └─► Infrastructure Dev
        - Ressources AWS (S3, Glue, Redshift, etc.)
        - Plus petit sizing (coûts réduits)
        - Peut être arrêté le soir/weekend

COMPTE STAGING
└─► Domaine SMUS: acme-staging-unified-domain
    │
    ├─► Projets Staging (miroir de prod)
    │   - finance-customer360-stg-project
    │   - sales-forecasting-stg-project
    │
    ├─► Données Staging
    │   - Copie anonymisée de prod
    │   - Données réalistes
    │   - Volume similaire à prod
    │
    └─► Infrastructure Staging
        - Configuration identique à prod
        - Sizing similaire à prod
        - Tests de charge

COMPTE PROD
└─► Domaine SMUS: acme-prod-unified-domain
    │
    ├─► Projets Prod
    │   - finance-customer360-prod-project
    │   - sales-forecasting-prod-project
    │
    ├─► Données Prod
    │   - DONNÉES RÉELLES
    │   - Clients réels
    │   - Conformité RGPD/SOC2
    │
    └─► Infrastructure Prod
        - Haute disponibilité
        - Backups automatiques
        - Monitoring 24/7
        - DR (Disaster Recovery)
```

---

## Flux de Travail Typique

```
DÉVELOPPEUR                    DEV ACCOUNT
     │
     │ 1. Code dans
     │    Git on-prem
     ▼
┌─────────────┐
│ GitLab      │
│ Self-Managed│──► 2. Push code
└─────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│ CI/CD Pipeline (CodePipeline)               │
│                                             │
│ 3. Build & Test                             │
│    ├─► Tests unitaires                      │
│    ├─► Tests d'intégration                  │
│    └─► Scan sécurité                        │
│                                             │
│ 4. Deploy automatique → DEV                 │
│    └─► acme-dev-unified-domain             │
└─────────────────────────────────────────────┘
     │
     │ 5. Tests OK en Dev
     ▼
┌─────────────────────────────────────────────┐
│ Approbation Manuelle (Chef de projet)      │
└─────────────────────────────────────────────┘
     │
     │ 6. Deploy → STAGING
     ▼
┌─────────────────────────────────────────────┐
│ STAGING Account                             │
│ └─► Tests UAT (User Acceptance Testing)    │
│ └─► Tests de charge                         │
│ └─► Validation métier                       │
└─────────────────────────────────────────────┘
     │
     │ 7. Validation OK
     ▼
┌─────────────────────────────────────────────┐
│ Approbation Manuelle (Ops Manager)         │
│ + Change Management                         │
└─────────────────────────────────────────────┘
     │
     │ 8. Deploy → PROD
     ▼
┌─────────────────────────────────────────────┐
│ PROD Account                                │
│ └─► Déploiement en prod                     │
│ └─► Monitoring actif                        │
│ └─► Alertes configurées                     │
└─────────────────────────────────────────────┘
```

---

## Exemple Concret

Imaginons votre équipe Finance qui crée un projet "Customer 360" :

### **Étape 1 : Développement (DEV Account)**
```
Compte: 123456789012 (DEV)
Domaine: acme-dev-unified-domain
Projet: finance-customer360-dev-project

Actions:
- Les data engineers créent le pipeline ETL
- Tests avec 1000 lignes de données test
- Expérimentation libre
- Coût: ~500€/mois
```

### **Étape 2 : Validation (STAGING Account)**
```
Compte: 234567890123 (STAGING)
Domaine: acme-staging-unified-domain
Projet: finance-customer360-stg-project

Actions:
- Tests avec 10 millions de lignes (données réalistes)
- Validation par le Product Owner Finance
- Tests de performance
- Coût: ~2000€/mois
```

### **Étape 3 : Production (PROD Account)**
```
Compte: 345678901234 (PROD)
Domaine: acme-prod-unified-domain
Projet: finance-customer360-prod-project

Actions:
- Données réelles clients (50M+ lignes)
- Utilisé par les analystes Finance
- SLA de 99.9% uptime
- Backups quotidiens
- Coût: ~8000€/mois
```

---

## Configuration AWS Organizations

```
┌────────────────────────────────────────────────────────┐
│ AWS Organizations - Structure Recommandée              │
├────────────────────────────────────────────────────────┤
│                                                         │
│  Root (Management Account)                             │
│  │                                                      │
│  ├─► OU: DataMesh                                      │
│  │   │                                                  │
│  │   ├─► OU: Dev                                       │
│  │   │   └─► Account: 123456789012 (dev)              │
│  │   │       - Tag: Environment=dev                    │
│  │   │       - Tag: CostCenter=DataPlatform           │
│  │   │                                                  │
│  │   ├─► OU: Staging                                   │
│  │   │   └─► Account: 234567890123 (staging)          │
│  │   │       - Tag: Environment=staging                │
│  │   │       - Tag: CostCenter=DataPlatform           │
│  │   │                                                  │
│  │   └─► OU: Prod                                      │
│  │       └─► Account: 345678901234 (prod)             │
│  │           - Tag: Environment=prod                   │
│  │           - Tag: CostCenter=DataPlatform           │
│  │                                                      │
│  └─► SCPs (Service Control Policies)                   │
│      - Prevent region usage outside eu-west-1          │
│      - Enforce encryption                              │
│      - Require MFA for sensitive actions               │
└────────────────────────────────────────────────────────┘
```

---

## Cross-Account Access (Accès Inter-Comptes)

Votre Management App doit pouvoir gérer les 3 comptes :

```
┌─────────────────────────────────────────────────┐
│ Management App (s'exécute dans un compte tools) │
└─────────────────────────────────────────────────┘
         │
         │ assume-role
         │
    ┌────┴────┬─────────┬─────────┐
    │         │         │         │
    ▼         ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐
│ Dev    │ │ Staging│ │ Prod   │
│ Role:  │ │ Role:  │ │ Role:  │
│ DataMesh│ DataMesh│ DataMesh│
│ Admin  │ │ Admin  │ │ ReadOnly│
└────────┘ └────────┘ └────────┘

Code Python:
# Connexion à Dev
dev_session = boto3.Session()
dev_client = dev_session.client(
    'sts'
).assume_role(
    RoleArn='arn:aws:iam::123456789012:role/DataMeshAdmin',
    RoleSessionName='mgmt-app'
)

# Créer client DataZone pour Dev
dz_dev = boto3.client(
    'datazone',
    aws_access_key_id=dev_client['Credentials']['AccessKeyId'],
    aws_secret_access_key=dev_client['Credentials']['SecretAccessKey'],
    aws_session_token=dev_client['Credentials']['SessionToken']
)

# Créer un projet dans Dev
dz_dev.create_project(
    domainIdentifier='acme-dev-unified-domain',
    name='finance-newproject-dev-project'
)
```

---

## Avantages Clés de Cette Architecture

✅ **Sécurité renforcée** - Blast radius limité
✅ **Conformité** - Prod isolé avec audit strict
✅ **Agilité Dev** - Les devs peuvent expérimenter sans risque
✅ **Coûts maîtrisés** - Visibilité par environnement
✅ **Promotion contrôlée** - Dev → Staging → Prod avec validations




---




# Qu'est-ce qu'un SMUS Domain ?

## SMUS = SageMaker Unified Studio

**SMUS Domain** signifie **SageMaker Unified Studio Domain** (parfois appelé aussi "Amazon DataZone Domain" dans les versions antérieures).

---

## Définition Simple

Un **SMUS Domain** est le **conteneur principal** qui regroupe :
- Vos utilisateurs
- Vos projets (projects)
- Vos données (assets)
- Vos catalogues
- Vos glossaires
- La gouvernance des données

C'est comme une **"plateforme Data Mesh complète"** dans un seul environnement isolé.

---

## Analogie

Pensez au SMUS Domain comme :

```
┌─────────────────────────────────────────────────────┐
│  SMUS Domain = Une "Entreprise Virtuelle"          │
│                                                      │
│  Tout comme une entreprise a:                       │
│  - Des départements (Domain Units)                  │
│  - Des projets (Projects)                           │
│  - Des employés (Users)                             │
│  - Des données (Assets)                             │
│  - Des règles (Policies)                            │
│                                                      │
│  Un SMUS Domain a exactement la même structure !    │
└─────────────────────────────────────────────────────┘
```

---

## Structure d'un SMUS Domain

```
┌──────────────────────────────────────────────────────────────┐
│  SMUS Domain: acme-prod-unified-domain                       │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  1. UTILISATEURS (Users & Groups)                      │ │
│  │     - finance-team (groupe)                            │ │
│  │     - sales-team (groupe)                              │ │
│  │     - data-engineers (groupe)                          │ │
│  │     - john.doe@acme.com (utilisateur)                  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  2. DOMAIN UNITS (Unités Organisationnelles)           │ │
│  │     Root Domain Unit                                   │ │
│  │     ├─► finance-domain-unit                            │ │
│  │     ├─► sales-domain-unit                              │ │
│  │     └─► operations-domain-unit                         │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  3. PROJECTS (Projets de données)                      │ │
│  │     - finance-customer360-prod-project                 │ │
│  │     - sales-forecasting-prod-project                   │ │
│  │     - ops-inventory-prod-project                       │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  4. CATALOG (Catalogue de données)                     │ │
│  │     - acme-prod-central-catalog                        │ │
│  │     - finance-prod-catalog                             │ │
│  │     - sales-prod-catalog                               │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  5. GLOSSARIES (Dictionnaires métier)                  │ │
│  │     - acme-enterprise-glossary                         │ │
│  │     - finance-glossary                                 │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  6. ASSETS (Données publiées)                          │ │
│  │     - Customer_Master_Data                             │ │
│  │     - Sales_Transactions                               │ │
│  │     - Product_Catalog                                  │ │
│  └────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

---

## Pourquoi un Domain par Compte ?

Dans votre architecture, vous avez **3 domaines SMUS** (un par environnement) :

```
┌─────────────────────────────────────────────────────┐
│  Compte DEV (123456789012)                          │
│  └─► acme-dev-unified-domain                        │
│      - Pour développement et tests                  │
│      - Données de test                              │
│      - Expérimentation                              │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  Compte STAGING (234567890123)                      │
│  └─► acme-staging-unified-domain                    │
│      - Pour validation pré-production               │
│      - Données anonymisées réalistes                │
│      - Tests UAT                                    │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  Compte PROD (345678901234)                         │
│  └─► acme-prod-unified-domain                       │
│      - Environnement de production                  │
│      - Données réelles                              │
│      - Utilisateurs finaux                          │
└─────────────────────────────────────────────────────┘
```

**Pourquoi ?** Parce que chaque environnement est **complètement isolé**. Les domaines ne peuvent pas communiquer entre eux directement.

---

## Ce que Contient un SMUS Domain

### 1. **Identité du Domain**

```
Domain ID: dzd_abc123xyz456
Domain Name: acme-prod-unified-domain
ARN: arn:aws:datazone:eu-west-1:345678901234:domain/dzd_abc123xyz456
Region: eu-west-1
Account: 345678901234
```

### 2. **Configuration**

```yaml
Domain Configuration:
  - Authentication: AWS IAM Identity Center (SSO)
  - Encryption: AWS KMS (Customer Managed Key)
  - VPC: vpc-abc123
  - Subnets: subnet-a, subnet-b, subnet-c
  - S3 Bucket: s3://acme-prod-datazone-bucket
```

### 3. **Ressources AWS Créées Automatiquement**

Quand vous créez un SMUS Domain, AWS crée automatiquement :

```
┌─────────────────────────────────────────────────┐
│ Ressources AWS Créées Automatiquement          │
├─────────────────────────────────────────────────┤
│                                                 │
│ • S3 Bucket (stockage des assets)              │
│ • AWS Glue Database (métadonnées)              │
│ • Lake Formation (permissions)                 │
│ • IAM Roles (rôles de service)                 │
│ • KMS Keys (si encryption custom)              │
│ • CloudWatch Logs (audit)                      │
│ • EventBridge Rules (événements)               │
└─────────────────────────────────────────────────┘
```

---

## Exemple Concret : Créer un SMUS Domain

### Via Console AWS :

```
1. Aller sur AWS Console
2. Chercher "SageMaker Unified Studio" ou "DataZone"
3. Cliquer "Create domain"
4. Remplir:
   - Name: acme-prod-unified-domain
   - Description: Production data mesh domain
   - Region: eu-west-1
   - Authentication: IAM Identity Center
   - VPC: vpc-prod-123
5. Créer
```

### Via Code Python (boto3) :

```python
import boto3

datazone = boto3.client('datazone', region_name='eu-west-1')

# Créer un domain
response = datazone.create_domain(
    name='acme-prod-unified-domain',
    description='Production data mesh domain for ACME Corp',
    domainExecutionRole='arn:aws:iam::345678901234:role/DataZoneDomainExecution',
    # Configuration SSO
    singleSignOn={
        'type': 'IAM_IDC',
        'userAssignment': 'AUTOMATIC'
    },
    # Tags
    tags={
        'Environment': 'prod',
        'CostCenter': 'DataPlatform',
        'ManagedBy': 'Terraform'
    }
)

domain_id = response['id']
print(f"Domain créé: {domain_id}")
```

---

## SMUS Domain vs AWS Account

```
┌─────────────────────────────────────────────────────┐
│  AWS Account (Compte AWS)                           │
│  - Limite de facturation                            │
│  - Limite de sécurité                               │
│  - Peut contenir PLUSIEURS services AWS             │
│                                                      │
│  ┌───────────────────────────────────────────────┐ │
│  │  SMUS Domain                                  │ │
│  │  - Limite organisationnelle pour data mesh   │ │
│  │  - Un domaine = une plateforme data          │ │
│  │  - Gère utilisateurs, projets, données       │ │
│  └───────────────────────────────────────────────┘ │
│                                                      │
│  ┌───────────────────────────────────────────────┐ │
│  │  Autres services AWS (optionnel)             │ │
│  │  - EC2, Lambda, RDS, etc.                    │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

**Un compte AWS** peut avoir :
- **1 seul SMUS Domain** (recommandé pour simplifier)
- **Plusieurs SMUS Domains** (pour des cas avancés - une domain par BU par exemple)

---

## Pourquoi "Unified Studio" ?

SageMaker Unified Studio unifie plusieurs outils AWS :

```
┌────────────────────────────────────────────────────┐
│  SMUS (SageMaker Unified Studio)                   │
│  = Plateforme unifiée qui combine:                 │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ Amazon DataZone                              │ │
│  │ (Data cataloging & governance)               │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ AWS Glue                                     │ │
│  │ (ETL & Data Catalog)                         │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ Amazon Athena                                │ │
│  │ (SQL queries)                                │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ Amazon Redshift                              │ │
│  │ (Data warehouse)                             │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ Amazon SageMaker                             │ │
│  │ (ML/AI)                                      │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │ Amazon EMR                                   │ │
│  │ (Big data processing)                        │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

Tout est accessible depuis **une seule interface web** = le portail SMUS !

---

## En Résumé

| Aspect | Description |
|--------|-------------|
| **Qu'est-ce que c'est ?** | Un environnement complet pour gérer votre data mesh |
| **Qui l'utilise ?** | Data engineers, Data analysts, Data scientists |
| **Que contient-il ?** | Users, Projects, Catalogs, Glossaries, Assets |
| **Combien par compte ?** | 1 domain recommandé (mais peut avoir plusieurs) |
| **Relation avec AWS** | 1 Domain existe dans 1 compte AWS, 1 région AWS |
| **Isolation** | Chaque domain est complètement isolé des autres |

---

# Architecture avec 3 Comptes PROD Séparés

Excellente approche ! Créer **3 comptes PROD distincts** offre une isolation maximale tout en gardant les données dans l'environnement de production.

---

## Architecture Recommandée : 3 Comptes PROD

```
┌─────────────────────────────────────────────────────────────────────────┐
│  AWS ORGANIZATION (Root Account)                                        │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  OU: Production                                                    │ │
│  │                                                                     │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │ │
│  │  │ PROD-DEV         │  │ PROD-STAGING     │  │ PROD             │ │ │
│  │  │ ACCOUNT          │  │ ACCOUNT          │  │ ACCOUNT          │ │ │
│  │  │ 111111111111     │  │ 222222222222     │  │ 333333333333     │ │ │
│  │  │                  │  │                  │  │                  │ │ │
│  │  │ Purpose:         │  │ Purpose:         │  │ Purpose:         │ │ │
│  │  │ Experimentation  │  │ Validation       │  │ Live Production  │ │ │
│  │  │ with PROD data   │  │ Pre-prod testing │  │ End users        │ │ │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘ │ │
│  │           │                      │                      │           │ │
│  │           ▼                      ▼                      ▼           │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │ │
│  │  │ SMUS Domain      │  │ SMUS Domain      │  │ SMUS Domain      │ │ │
│  │  │ acme-prod-dev    │  │ acme-prod-stg    │  │ acme-prod        │ │ │
│  │  │ -unified-domain  │  │ -unified-domain  │  │ -unified-domain  │ │ │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  OU: Development (Optional - pour dev de la management app)       │ │
│  │                                                                     │ │
│  │  ┌──────────────────┐                                              │ │
│  │  │ DEV ACCOUNT      │  Pour développer la management app          │ │
│  │  │ 444444444444     │  et le CI/CD pipeline                       │ │
│  │  └──────────────────┘                                              │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Détail de Chaque Compte PROD

### 🔵 COMPTE PROD-DEV (111111111111)

```
┌────────────────────────────────────────────────────────────────────┐
│  PROD-DEV ACCOUNT                                                  │
│  Purpose: Expérimentation sécurisée avec données de production    │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  SMUS Domain: acme-prod-dev-unified-domain                   │ │
│  │                                                               │ │
│  │  Domain Units:                                                │ │
│  │  ├─► finance-domain-unit                                      │ │
│  │  ├─► sales-domain-unit                                        │ │
│  │  └─► ops-domain-unit                                          │ │
│  │                                                               │ │
│  │  Projects:                                                    │ │
│  │  ├─► finance-customer360-prod-dev-project                     │ │
│  │  ├─► sales-forecasting-prod-dev-project                       │ │
│  │  └─► ops-inventory-prod-dev-project                           │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  DATA STRATEGY                                               │ │
│  │                                                               │ │
│  │  Option A: Read-Only Access to PROD Account                  │ │
│  │  ──────────────────────────────────────────────              │ │
│  │  • AWS Lake Formation cross-account grants                   │ │
│  │  • READ ONLY permissions sur PROD data                       │ │
│  │  • Row-level filtering (sample 10%)                          │ │
│  │  • Column-level security (mask PII)                          │ │
│  │                                                               │ │
│  │  Option B: Replicated Sample Data                            │ │
│  │  ──────────────────────────────────────                      │ │
│  │  • Scheduled replication from PROD (daily/weekly)            │ │
│  │  • 10% sample of production data                             │ │
│  │  • PII masked automatically                                  │ │
│  │  • Stored in: s3://acme-prod-dev-data/                       │ │
│  │                                                               │ │
│  │  Own Storage (for experimentation outputs):                  │ │
│  │  s3://acme-prod-dev-sandbox/                                 │ │
│  │  ├─► finance-customer360/                                    │ │
│  │  │   ├─► notebooks/                                          │ │
│  │  │   ├─► queries/                                            │ │
│  │  │   └─► experiments/                                        │ │
│  │  └─► shared/                                                 │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  ACCESS CONTROL                                              │ │
│  │                                                               │ │
│  │  Who can access:                                             │ │
│  │  • Data Analysts (full access)                               │ │
│  │  • Data Engineers (full access)                              │ │
│  │  • Business Users (read access)                              │ │
│  │  • Data Scientists (full access)                             │ │
│  │                                                               │ │
│  │  Restrictions:                                               │ │
│  │  • Cannot modify PROD source data                            │ │
│  │  • Can create/delete own resources                           │ │
│  │  • Budget limit: 5,000 EUR/month                             │ │
│  │  • Auto-shutdown non-critical resources at night             │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

### 🟡 COMPTE PROD-STAGING (222222222222)

```
┌────────────────────────────────────────────────────────────────────┐
│  PROD-STAGING ACCOUNT                                              │
│  Purpose: Validation pré-production avec données complètes         │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  SMUS Domain: acme-prod-staging-unified-domain               │ │
│  │                                                               │ │
│  │  Projects:                                                    │ │
│  │  ├─► finance-customer360-prod-stg-project                     │ │
│  │  ├─► sales-forecasting-prod-stg-project                       │ │
│  │  └─► ops-inventory-prod-stg-project                           │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  DATA STRATEGY                                               │ │
│  │                                                               │ │
│  │  Option A: Read Access to PROD Account                       │ │
│  │  ───────────────────────────────────────                     │ │
│  │  • Lake Formation cross-account READ                         │ │
│  │  • Access to 100% of PROD data                               │ │
│  │  • NO row/column filtering                                   │ │
│  │  • Real-time access to latest data                           │ │
│  │                                                               │ │
│  │  Option B: Daily Full Replication                            │ │
│  │  ──────────────────────────────                              │ │
│  │  • Automated daily sync from PROD                            │ │
│  │  • 100% of production data                                   │ │
│  │  • Stored in: s3://acme-prod-staging-data/                   │ │
│  │  • Maintains data freshness                                  │ │
│  │                                                               │ │
│  │  Own Storage (for validation outputs):                       │ │
│  │  s3://acme-prod-staging-outputs/                             │ │
│  │  ├─► validation-results/                                     │ │
│  │  ├─► performance-tests/                                      │ │
│  │  └─► integration-tests/                                      │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  ACCESS CONTROL                                              │ │
│  │                                                               │ │
│  │  Who can access:                                             │ │
│  │  • Product Owners (validation)                               │ │
│  │  • QA Team (testing)                                         │ │
│  │  • Senior Data Engineers (troubleshooting)                   │ │
│  │  • CI/CD Pipeline (automated deployments)                    │ │
│  │                                                               │ │
│  │  Restrictions:                                               │ │
│  │  • Cannot modify PROD source data                            │ │
│  │  • Can run performance/load tests                            │ │
│  │  • Manual approval required for changes                      │ │
│  │  • Budget limit: 15,000 EUR/month                            │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

### 🔴 COMPTE PROD (333333333333)

```
┌────────────────────────────────────────────────────────────────────┐
│  PROD ACCOUNT (LIVE PRODUCTION)                                    │
│  Purpose: Production live pour utilisateurs finaux                 │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  SMUS Domain: acme-prod-unified-domain                       │ │
│  │                                                               │ │
│  │  Projects:                                                    │ │
│  │  ├─► finance-customer360-prod-project                         │ │
│  │  ├─► sales-forecasting-prod-project                           │ │
│  │  └─► ops-inventory-prod-project                               │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  DATA STRATEGY                                               │ │
│  │                                                               │ │
│  │  SOURCE OF TRUTH - Production Data                           │ │
│  │  ────────────────────────────────                            │ │
│  │  • All real production data                                  │ │
│  │  • Customer data (PII protected)                             │ │
│  │  • Transactional data                                        │ │
│  │  • Business critical data                                    │ │
│  │                                                               │ │
│  │  Storage:                                                     │ │
│  │  s3://acme-prod-data/                                        │ │
│  │  ├─► raw/ (source of truth)                                  │ │
│  │  ├─► curated/ (certified data products)                      │ │
│  │  └─► published/ (assets in catalog)                          │ │
│  │                                                               │ │
│  │  Shared via Lake Formation to:                               │ │
│  │  • PROD-DEV account (READ, sampled, masked)                  │ │
│  │  • PROD-STAGING account (READ, full access)                  │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  ACCESS CONTROL (MOST RESTRICTIVE)                           │ │
│  │                                                               │ │
│  │  Who can access:                                             │ │
│  │  • Business Analysts (READ ONLY)                             │ │
│  │  • Executives (Dashboards only)                              │ │
│  │  • Applications (via service accounts)                       │ │
│  │  • Operations Team (emergency access)                        │ │
│  │                                                               │ │
│  │  Restrictions:                                               │ │
│  │  • NO direct modification by users                           │ │
│  │  • All changes via CI/CD only                                │ │
│  │  • MFA required for console access                           │ │
│  │  • All actions logged to CloudTrail                          │ │
│  │  • Change Management required                                │ │
│  │  • SLA: 99.9% uptime                                         │ │
│  │  • 24/7 monitoring and alerts                                │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

---

## Cross-Account Data Sharing Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│  AWS LAKE FORMATION - CROSS-ACCOUNT DATA SHARING                   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  PROD ACCOUNT (333333333333) - Data Owner                    │ │
│  │                                                               │ │
│  │  S3 Location: s3://acme-prod-data/raw/customers/             │ │
│  │  Glue Database: prod_customers_db                            │ │
│  │  Glue Table: customers                                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                │                            │                      │
│                │ Grant READ                 │ Grant READ           │
│                │ + Row Filter (10%)         │ + Full Access        │
│                │ + Column Mask (PII)        │                      │
│                │                            │                      │
│                ▼                            ▼                      │
│  ┌─────────────────────────┐    ┌─────────────────────────┐      │
│  │ PROD-DEV ACCOUNT        │    │ PROD-STAGING ACCOUNT    │      │
│  │ (111111111111)          │    │ (222222222222)          │      │
│  │                         │    │                         │      │
│  │ Resource Share:         │    │ Resource Share:         │      │
│  │ • READ permissions      │    │ • READ permissions      │      │
│  │ • Row filter: 10%       │    │ • All rows              │      │
│  │ • Columns: masked PII   │    │ • All columns           │      │
│  │                         │    │                         │      │
│  │ Projects can query:     │    │ Projects can query:     │      │
│  │ SELECT * FROM           │    │ SELECT * FROM           │      │
│  │ shared_customers        │    │ shared_customers        │      │
│  │ --> 10% sample          │    │ --> 100% data           │      │
│  └─────────────────────────┘    └─────────────────────────┘      │
└────────────────────────────────────────────────────────────────────┘
```

---

## Configuration Lake Formation Cross-Account

### Dans PROD Account (333333333333) - Data Owner

```python
import boto3

lf = boto3.client('lakeformation', region_name='eu-west-1')
ram = boto3.client('ram', region_name='eu-west-1')

# ============================================================
# ÉTAPE 1: Activer cross-account sharing dans PROD
# ============================================================

# Enregistrer le S3 location
lf.register_resource(
    ResourceArn='arn:aws:s3:::acme-prod-data',
    UseServiceLinkedRole=True
)

# ============================================================
# ÉTAPE 2: Créer Resource Share pour PROD-DEV
# ============================================================

# Partager la Glue Database avec PROD-DEV account
ram_share_dev = ram.create_resource_share(
    name='prod-to-dev-data-share',
    resourceArns=[
        'arn:aws:glue:eu-west-1:333333333333:database/prod_customers_db',
        'arn:aws:glue:eu-west-1:333333333333:table/prod_customers_db/customers'
    ],
    principals=[
        '111111111111'  # PROD-DEV account ID
    ],
    allowExternalPrincipals=False,
    tags=[
        {'key': 'Environment', 'value': 'prod-dev-share'},
        {'key': 'Purpose', 'value': 'development-experimentation'}
    ]
)

# ============================================================
# ÉTAPE 3: Grant permissions à PROD-DEV avec restrictions
# ============================================================

lf.grant_permissions(
    Principal={
        'DataLakePrincipalIdentifier': '111111111111'  # PROD-DEV account
    },
    Resource={
        'Table': {
            'CatalogId': '333333333333',
            'DatabaseName': 'prod_customers_db',
            'Name': 'customers'
        }
    },
    Permissions=['SELECT', 'DESCRIBE'],
    PermissionsWithGrantOption=[],
    # Data Filters
    DataFilters={
        'Name': 'dev_sample_and_mask',
        'RowFilter': {
            'FilterExpression': 'MOD(hash(customer_id), 10) = 0'  # 10% sample
        },
        'ColumnWildcard': {
            'ExcludedColumnNames': [
                'ssn',
                'credit_card_number',
                'bank_account',
                'salary',
                'date_of_birth'
            ]
        }
    }
)

# ============================================================
# ÉTAPE 4: Créer Resource Share pour PROD-STAGING
# ============================================================

ram_share_stg = ram.create_resource_share(
    name='prod-to-staging-data-share',
    resourceArns=[
        'arn:aws:glue:eu-west-1:333333333333:database/prod_customers_db',
        'arn:aws:glue:eu-west-1:333333333333:table/prod_customers_db/customers'
    ],
    principals=[
        '222222222222'  # PROD-STAGING account ID
    ],
    allowExternalPrincipals=False
)

# ============================================================
# ÉTAPE 5: Grant permissions à PROD-STAGING (full access)
# ============================================================

lf.grant_permissions(
    Principal={
        'DataLakePrincipalIdentifier': '222222222222'  # PROD-STAGING account
    },
    Resource={
        'Table': {
            'CatalogId': '333333333333',
            'DatabaseName': 'prod_customers_db',
            'Name': 'customers'
        }
    },
    Permissions=['SELECT', 'DESCRIBE'],
    PermissionsWithGrantOption=[]
    # No filters - full access
)

print("Cross-account sharing configured successfully!")
```

### Dans PROD-DEV Account (111111111111) - Consumer

```python
import boto3

lf = boto3.client('lakeformation', region_name='eu-west-1')
ram = boto3.client('ram', region_name='eu-west-1')

# ============================================================
# ÉTAPE 1: Accepter le Resource Share
# ============================================================

# Lister les invitations
invitations = ram.get_resource_share_invitations(
    resourceShareArns=[
        'arn:aws:ram:eu-west-1:333333333333:resource-share/xxxx'
    ]
)

# Accepter
for invitation in invitations['resourceShareInvitations']:
    ram.accept_resource_share_invitation(
        resourceShareInvitationArn=invitation['resourceShareInvitationArn']
    )

# ============================================================
# ÉTAPE 2: Créer Resource Link (permet d'utiliser la table)
# ============================================================

glue = boto3.client('glue', region_name='eu-west-1')

glue.create_database(
    DatabaseInput={
        'Name': 'shared_from_prod',
        'Description': 'Shared data from PROD account',
        'TargetDatabase': {
            'CatalogId': '333333333333',
            'DatabaseName': 'prod_customers_db'
        }
    }
)

# ============================================================
# ÉTAPE 3: Grant permissions au SMUS Domain dans PROD-DEV
# ============================================================

lf.grant_permissions(
    Principal={
        'DataLakePrincipalIdentifier': 'arn:aws:iam::111111111111:role/AmazonDataZone-acme-prod-dev-domain'
    },
    Resource={
        'Table': {
            'CatalogId': '333333333333',  # Source catalog (PROD)
            'DatabaseName': 'prod_customers_db',
            'Name': 'customers'
        }
    },
    Permissions=['SELECT', 'DESCRIBE']
)

print("Resource share accepted and configured in PROD-DEV!")
```

---

## Flux de Promotion Multi-Comptes

```
┌────────────────────────────────────────────────────────────────┐
│  WORKFLOW: Project Lifecycle Across 3 PROD Accounts           │
└────────────────────────────────────────────────────────────────┘

PHASE 1: EXPERIMENTATION (PROD-DEV Account)
┌──────────────────────────────────────────────────────────────┐
│ Account: 111111111111 (PROD-DEV)                             │
│ Domain: acme-prod-dev-unified-domain                         │
│ Project: finance-customer360-prod-dev-project                │
│                                                               │
│ Actions:                                                      │
│ 1. Data Analyst creates project                             │
│ 2. Experiments with SQL queries                              │
│ 3. Builds ETL pipelines                                      │
│ 4. Creates dashboards                                        │
│ 5. Uses 10% sample of PROD data (PII masked)                │
│                                                               │
│ Git Repository: feature/customer360-v1                       │
│ - notebooks/customer_analysis.ipynb                          │
│ - queries/customer_360_view.sql                              │
│ - etl/transform_customer_data.py                             │
│                                                               │
│ Duration: 2-4 weeks                                          │
└──────────────────────────────────────────────────────────────┘
                           │
                           │ 1. Git Push to on-prem GitLab
                           │ 2. Code Review (MR approved)
                           │ 3. CI/CD triggered
                           ▼
┌──────────────────────────────────────────────────────────────┐
│ CI/CD PIPELINE - Promotion to STAGING                        │
│                                                               │
│ Steps:                                                        │
│ 1. Run automated tests                                       │
│ 2. Security scan (SAST/DAST)                                │
│ 3. Create project in PROD-STAGING account                   │
│ 4. Deploy code via CloudFormation                           │
│ 5. Configure Lake Formation permissions                     │
│ 6. Notify QA team                                           │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
PHASE 2: VALIDATION (PROD-STAGING Account)
┌──────────────────────────────────────────────────────────────┐
│ Account: 222222222222 (PROD-STAGING)                        │
│ Domain: acme-prod-staging-unified-domain                     │
│ Project: finance-customer360-prod-stg-project                │
│                                                               │
│ Actions:                                                      │
│ 1. QA team validates functionality                           │
│ 2. Product Owner reviews results                            │
│ 3. Performance testing (100% data volume)                   │
│ 4. Integration testing                                       │
│ 5. User Acceptance Testing (UAT)                            │
│                                                               │
│ Tests:                                                        │
│ - Functional tests: PASS                                     │
│ - Performance tests: Query time < 5s                         │
│ - Data quality: 99.5% accuracy                              │
│ - UAT approval: Product Owner signed off                    │
│                                                               │
│ Duration: 1-2 weeks                                          │
└──────────────────────────────────────────────────────────────┘
                           │
                           │ 1. All tests PASSED
                           │ 2. Product Owner approval
                           │ 3. Change Management ticket created
                           │ 4. Manual approval required
                           ▼
┌──────────────────────────────────────────────────────────────┐
│ CI/CD PIPELINE - Promotion to PRODUCTION                     │
│                                                               │
│ Steps:                                                        │
│ 1. Change Management approval (CAB)                         │
│ 2. Schedule deployment window                               │
│ 3. Create project in PROD account                           │
│ 4. Deploy code via CloudFormation                           │
│ 5. Configure monitoring & alerts                            │
│ 6. Smoke tests                                              │
│ 7. Rollback plan ready                                      │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
PHASE 3: PRODUCTION (PROD Account)
┌──────────────────────────────────────────────────────────────┐
│ Account: 333333333333 (PROD)                                │
│ Domain: acme-prod-unified-domain                             │
│ Project: finance-customer360-prod-project                    │
│                                                               │
│ Live Production:                                             │
│ - 500+ business users accessing                             │
│ - Real-time dashboards                                       │
│ - API integrations                                           │
│ - SLA: 99.9% uptime                                         │
│ - 24/7 monitoring                                            │
│                                                               │
│ Monitoring:                                                   │
│ - CloudWatch dashboards                                      │
│ - PagerDuty alerts                                           │
│ - Daily data quality checks                                  │
│ - Weekly usage reports                                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Management App - Multi-Account Operations

```python
# management_app/multi_account_manager.py

import boto3
from typing import Dict, List

class MultiAccountDataMeshManager:
    """
    Gère les opérations Data Mesh à travers 3 comptes PROD
    """
    
    def __init__(self):
        self.accounts = {
            'prod-dev': {
                'account_id': '111111111111',
                'domain_id': 'dzd_prod_dev_abc123',
                'role_arn': 'arn:aws:iam::111111111111:role/DataMeshAdmin'
            },
            'prod-staging': {
                'account_id': '222222222222',
                'domain_id': 'dzd_prod_stg_def456',
                'role_arn': 'arn:aws:iam::222222222222:role/DataMeshAdmin'
            },
            'prod': {
                'account_id': '333333333333',
                'domain_id': 'dzd_prod_ghi789',
                'role_arn': 'arn:aws:iam::333333333333:role/DataMeshAdmin'
            }
        }
    
    def get_session(self, account_env: str):
        """Obtenir une session boto3 pour un compte spécifique"""
        sts = boto3.client('sts')
        
        assumed_role = sts.assume_role(
            RoleArn=self.accounts[account_env]['role_arn'],
            RoleSessionName=f'datamesh-mgmt-{account_env}'
        )
        
        return boto3.Session(
            aws_access_key_id=assumed_role['Credentials']['AccessKeyId'],
            aws_secret_access_key=assumed_role['Credentials']['SecretAccessKey'],
            aws_session_token=assumed_role['Credentials']['SessionToken']
        )
    
    def promote_project(self, project_name: str, from_env: str, to_env: str):
        """
        Promouvoir un projet d'un environnement à un autre
        
        Args:
            project_name: finance-customer360
            from_env: 'prod-dev'
            to_env: 'prod-staging'
        """
        print(f"Promoting {project_name} from {from_env} to")

------


# Architecture de Promotion de Projets entre 3 Comptes PROD

## Vue d'Ensemble

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLUX DE PROMOTION                                 │
│                                                                      │
│  PROD-DEV          →        PROD-STAGING      →       PROD          │
│  (Experimentation)          (Validation)              (Live)        │
│                                                                      │
│  2-4 semaines              1-2 semaines              ∞              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. ARCHITECTURE DES 3 COMPTES

```
┌────────────────────────────────────────────────────────────────────┐
│  PROD-DEV ACCOUNT (111111111111)                                   │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SMUS Domain: acme-prod-dev-unified-domain                         │
│                                                                     │
│  PROJETS EN DÉVELOPPEMENT:                                         │
│  ├─► finance-customer360-prod-dev-project     [Status: Active]    │
│  ├─► sales-forecasting-prod-dev-project       [Status: Active]    │
│  └─► marketing-segmentation-prod-dev-project  [Status: Draft]     │
│                                                                     │
│  STOCKAGE:                                                          │
│  ├─► S3: s3://acme-prod-dev-artifacts/                            │
│  │   ├─► projects/finance-customer360/                            │
│  │   │   ├─► queries/                                             │
│  │   │   ├─► notebooks/                                           │
│  │   │   ├─► etl-scripts/                                         │
│  │   │   └─► config/                                              │
│  │   └─► shared/                                                  │
│  │                                                                 │
│  └─► Glue Catalog: shared_from_prod (resource link)               │
│      └─► Access: READ 10% sampled + PII masked                    │
│                                                                     │
│  GIT INTEGRATION:                                                  │
│  └─► Branch Strategy:                                              │
│      ├─► feature/* (développement actif)                          │
│      ├─► develop (intégration)                                    │
│      └─► release/staging (prêt pour staging)                      │
└────────────────────────────────────────────────────────────────────┘

                              ▼
                    [PROMOTION PROCESS]
                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  PROD-STAGING ACCOUNT (222222222222)                               │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SMUS Domain: acme-prod-staging-unified-domain                     │
│                                                                     │
│  PROJETS EN VALIDATION:                                            │
│  ├─► finance-customer360-prod-stg-project     [Status: Testing]   │
│  └─► sales-forecasting-prod-stg-project       [Status: UAT]       │
│                                                                     │
│  STOCKAGE:                                                          │
│  ├─► S3: s3://acme-prod-staging-artifacts/                        │
│  │   ├─► projects/finance-customer360/                            │
│  │   │   ├─► test-results/                                        │
│  │   │   ├─► performance-metrics/                                 │
│  │   │   ├─► validation-reports/                                  │
│  │   │   └─► deployed-config/                                     │
│  │   └─► shared/                                                  │
│  │                                                                 │
│  └─► Glue Catalog: shared_from_prod (resource link)               │
│      └─► Access: READ 100% full data                              │
│                                                                     │
│  GIT INTEGRATION:                                                  │
│  └─► Branch Strategy:                                              │
│      ├─► release/staging (déployé ici)                            │
│      └─► release/production (prêt pour prod)                      │
└────────────────────────────────────────────────────────────────────┘

                              ▼
                    [PROMOTION PROCESS]
                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  PROD ACCOUNT (333333333333)                                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  SMUS Domain: acme-prod-unified-domain                             │
│                                                                     │
│  PROJETS EN PRODUCTION:                                            │
│  ├─► finance-customer360-prod-project         [Status: Live]      │
│  │   └─► Users: 500+ business users                               │
│  └─► sales-forecasting-prod-project           [Status: Live]      │
│      └─► Users: 200+ sales analysts                               │
│                                                                     │
│  STOCKAGE:                                                          │
│  ├─► S3: s3://acme-prod-data/                                     │
│  │   ├─► raw/ (SOURCE OF TRUTH)                                   │
│  │   ├─► curated/                                                 │
│  │   └─► published/                                               │
│  │                                                                 │
│  └─► Glue Catalog: prod_data_catalog (MASTER)                     │
│      └─► Shared to PROD-DEV & PROD-STAGING via Lake Formation     │
│                                                                     │
│  GIT INTEGRATION:                                                  │
│  └─► Branch Strategy:                                              │
│      ├─► main (production stable)                                 │
│      └─► hotfix/* (emergency fixes)                               │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. STRATÉGIE DE PROMOTION DÉTAILLÉE

### Phase 1: PROD-DEV → PROD-STAGING

```
┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 1: PRÉPARATION dans PROD-DEV                                │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Checklist Avant Promotion:                                        │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ □ Code validé et testé en dev                                │ │
│  │ □ Documentation à jour (README, configs)                     │ │
│  │ □ Tous les artefacts versionnés dans Git                     │ │
│  │ □ Metadata forms définies                                    │ │
│  │ □ Permissions mappées                                        │ │
│  │ □ Assets inventoriés                                         │ │
│  │ □ Dépendances documentées                                    │ │
│  │ □ Tests unitaires passent                                    │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Actions Manuelles:                                                │
│  1. Data Engineer crée un Merge Request dans GitLab               │
│     • Source: feature/customer360-v1                              │
│     • Target: release/staging                                     │
│     • Reviewers: Senior Data Engineers                            │
│                                                                     │
│  2. Code Review Process                                            │
│     • Reviewer vérifie la qualité du code                         │
│     • Reviewer vérifie les bonnes pratiques                       │
│     • Reviewer approuve le MR                                     │
│                                                                     │
│  3. Merge vers release/staging                                     │
│     • Trigger automatique du pipeline CI/CD                       │
└────────────────────────────────────────────────────────────────────┘

                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 2: CI/CD AUTOMATIQUE (Pipeline GitLab/Jenkins)              │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Pipeline Stages:                                                  │
│                                                                     │
│  Stage 1: BUILD & TEST                                             │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • Checkout code from Git                                     │ │
│  │ • Run linters (Python: pylint, SQL: sqlfluff)                │ │
│  │ • Run unit tests                                             │ │
│  │ • Build Docker images (si applicable)                        │ │
│  │ • Scan sécurité (Snyk, SonarQube)                           │ │
│  │ • Generate artifacts (packaged code)                         │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Stage 2: INFRASTRUCTURE PREPARATION                               │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • Assume role vers PROD-STAGING account                      │ │
│  │ • Vérifier que le domain existe                              │ │
│  │ • Vérifier les permissions Lake Formation                    │ │
│  │ • Créer/updater le projet SMUS via API                       │ │
│  │   └─► API: create_project() ou update_project()             │ │
│  │ • Créer les domain units si nécessaire                       │ │
│  │ • Configurer les blueprints requis                           │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Stage 3: DEPLOYMENT                                               │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • Upload artefacts vers S3 staging:                          │ │
│  │   s3://acme-prod-staging-artifacts/projects/customer360/     │ │
│  │                                                               │ │
│  │ • Deploy via CloudFormation/Terraform:                       │ │
│  │   ├─► Glue Jobs                                              │ │
│  │   ├─► Lambda Functions                                       │ │
│  │   ├─► Athena Named Queries                                   │ │
│  │   ├─► Redshift Views/Procedures                              │ │
│  │   └─► EventBridge Rules                                      │ │
│  │                                                               │ │
│  │ • Configurer Lake Formation permissions                      │ │
│  │ • Ajouter les membres au projet                              │ │
│  │ • Créer les assets initiaux dans l'inventaire                │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Stage 4: POST-DEPLOYMENT VERIFICATION                             │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • Run smoke tests                                            │ │
│  │ • Verify project is accessible                               │ │
│  │ • Check data permissions                                     │ │
│  │ • Notify QA team via Slack/Email                            │ │
│  │ • Update Jira ticket status → "Ready for Testing"           │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘

                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 3: VALIDATION dans PROD-STAGING                             │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Responsables: QA Team + Product Owner                             │
│  Durée: 1-2 semaines                                               │
│                                                                     │
│  Tests à Effectuer:                                                │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ FUNCTIONAL TESTING (QA Team)                                 │ │
│  │ • Toutes les queries retournent les résultats attendus       │ │
│  │ • Les dashboards s'affichent correctement                    │ │
│  │ • Les ETL jobs s'exécutent sans erreur                       │ │
│  │ • Les transformations produisent la data attendue            │ │
│  │                                                               │ │
│  │ PERFORMANCE TESTING (Data Engineers)                         │ │
│  │ • Query performance < 5 secondes                             │ │
│  │ • ETL completion time acceptable                             │ │
│  │ • Resource utilization optimale                              │ │
│  │ • Load testing avec 100% du volume de data                   │ │
│  │                                                               │ │
│  │ INTEGRATION TESTING (QA Team)                                │ │
│  │ • Intégration avec autres projets                            │ │
│  │ • Subscriptions fonctionnent                                 │ │
│  │ • Data lineage est correct                                   │ │
│  │ • Metadata est complet                                       │ │
│  │                                                               │ │
│  │ USER ACCEPTANCE TESTING (Product Owner + Business Users)     │ │
│  │ • Business logic est correcte                                │ │
│  │ • Résultats correspondent aux attentes métier                │ │
│  │ • UX/UI acceptable                                           │ │
│  │ • Sign-off formel du Product Owner                           │ │
│  │                                                               │ │
│  │ DATA QUALITY TESTING (Data Quality Team)                     │ │
│  │ • Completeness checks                                        │ │
│  │ • Accuracy validation                                        │ │
│  │ • Consistency verification                                   │ │
│  │ • Timeliness assessment                                      │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Documentation des Résultats:                                      │
│  • Test reports stockés dans S3                                   │
│  • Screenshots des dashboards                                     │
│  • Performance metrics loggés                                     │
│  • Issues trackés dans Jira                                       │
│                                                                     │
│  Critères de Succès:                                              │
│  ✓ 100% des tests fonctionnels passent                           │
│  ✓ Performance acceptable                                         │
│  ✓ 0 bugs critiques                                              │
│  ✓ Product Owner sign-off                                        │
│  ✓ Documentation complète                                         │
└────────────────────────────────────────────────────────────────────┘
```

---

### Phase 2: PROD-STAGING → PROD

```
┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 1: APPROBATION & PLANIFICATION                              │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Change Management Process:                                        │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ 1. CREATE CHANGE TICKET (ServiceNow/Jira)                    │ │
│  │    • Titre: Deploy Customer360 to Production                 │ │
│  │    • Type: Standard Change                                   │ │
│  │    • Risk Level: Medium                                      │ │
│  │    • Impact: 500+ users                                      │ │
│  │    • Downtime: None (zero-downtime deployment)               │ │
│  │                                                               │ │
│  │ 2. ATTACHMENTS                                               │ │
│  │    • Test results from STAGING                               │ │
│  │    • Deployment plan                                         │ │
│  │    • Rollback plan                                           │ │
│  │    • Communication plan                                      │ │
│  │    • Risk assessment                                         │ │
│  │                                                               │ │
│  │ 3. APPROVALS REQUIRED                                        │ │
│  │    □ Product Owner                                           │ │
│  │    □ Data Platform Manager                                   │ │
│  │    □ IT Operations Manager                                   │ │
│  │    □ Security Team (pour données sensibles)                  │ │
│  │    □ CAB (Change Advisory Board) - si high risk              │ │
│  │                                                               │ │
│  │ 4. DEPLOYMENT WINDOW                                         │ │
│  │    • Date: Mardi 15:00 - 17:00 CET                          │ │
│  │    • Raison: Low-traffic period                              │ │
│  │    • On-call: Data Platform Team                             │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Pre-Deployment Checklist:                                         │
│  □ All approvals obtained                                         │
│  □ Deployment window scheduled                                    │
│  □ Stakeholders notified                                          │
│  □ Rollback plan ready                                            │
│  □ Monitoring dashboards prepared                                 │
│  □ On-call team briefed                                           │
└────────────────────────────────────────────────────────────────────┘

                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 2: DÉPLOIEMENT EN PRODUCTION                                │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Deployment Strategy: BLUE-GREEN DEPLOYMENT                        │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │ CURRENT STATE (BLUE)                                       │   │
│  │ finance-customer360-prod-project v1.5 (LIVE)              │   │
│  │ └─► Serving 500 users                                     │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Déploiement Parallèle:                                            │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │ NEW DEPLOYMENT (GREEN)                                     │   │
│  │                                                             │   │
│  │ 1. Create new project: finance-customer360-prod-v2        │   │
│  │                                                             │   │
│  │ 2. Deploy all resources (via CI/CD):                       │   │
│  │    • Glue Jobs v2.0                                        │   │
│  │    • Athena Queries v2.0                                   │   │
│  │    • Dashboards v2.0                                       │   │
│  │    • Permissions configured                                │   │
│  │                                                             │   │
│  │ 3. Run smoke tests on v2                                   │   │
│  │    ✓ Queries work                                          │   │
│  │    ✓ Data accessible                                       │   │
│  │    ✓ Dashboards load                                       │   │
│  │                                                             │   │
│  │ 4. Canary Testing (10% of users)                          │   │
│  │    • Move 50 users to v2                                   │   │
│  │    • Monitor for 30 minutes                                │   │
│  │    • Check error rates                                     │   │
│  │    • Collect user feedback                                 │   │
│  │                                                             │   │
│  │ 5. If canary OK → Full cutover                            │   │
│  │    • Update project pointer: v1 → v2                       │   │
│  │    • All users now on v2                                   │   │
│  │    • Keep v1 running for 24h (rollback safety)            │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Monitoring During Deployment:                                     │
│  • CloudWatch metrics (every 1 minute)                             │
│  • Error logs streaming                                            │
│  • User activity tracking                                          │
│  • Performance metrics                                             │
│  • Data quality checks                                             │
└────────────────────────────────────────────────────────────────────┘

                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 3: POST-DEPLOYMENT VALIDATION                               │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Immediate Checks (0-1 hour):                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ □ All users can access the project                          │ │
│  │ □ Queries execute successfully                              │ │
│  │ □ Dashboards load correctly                                 │ │
│  │ □ No error spikes in logs                                   │ │
│  │ □ Performance within acceptable range                       │ │
│  │ □ Data freshness verified                                   │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  24-Hour Monitoring:                                               │
│  • Monitor CloudWatch alarms                                       │
│  • Review user feedback/tickets                                   │
│  • Check data quality metrics                                     │
│  • Verify ETL job completions                                     │
│  • Compare performance vs. baseline                               │
│                                                                     │
│  Week 1 Review:                                                    │
│  • Analyze usage patterns                                         │
│  • Review incident tickets                                        │
│  • Collect user satisfaction                                      │
│  • Assess business value delivered                                │
│                                                                     │
│  Success Criteria:                                                 │
│  ✓ Zero critical incidents                                        │
│  ✓ User adoption > 80%                                            │
│  ✓ Performance SLA met                                            │
│  ✓ Positive user feedback                                         │
│  ✓ Business metrics improved                                      │
└────────────────────────────────────────────────────────────────────┘

                              ▼

┌────────────────────────────────────────────────────────────────────┐
│  ÉTAPE 4: CLEANUP & DOCUMENTATION                                  │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Après 7 jours de succès:                                         │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • Decommission old version (v1)                              │ │
│  │ • Archive old artefacts                                      │ │
│  │ • Update documentation                                       │ │
│  │ • Close change ticket                                        │ │
│  │ • Conduct post-mortem (lessons learned)                     │ │
│  │ • Update runbooks                                            │ │
│  │ • Communicate success to stakeholders                        │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

---

## 3. ARTEFACTS À PROMOUVOIR

```
┌────────────────────────────────────────────────────────────────────┐
│  QUOI PROMOUVOIR entre les environnements ?                        │
└────────────────────────────────────────────────────────────────────┘

VERSIONNÉS DANS GIT (Source Control):
┌──────────────────────────────────────────────────────────────────┐
│ • Queries SQL (.sql files)                                       │
│ • ETL Scripts (Python, Spark, etc.)                              │
│ • Notebooks (.ipynb)                                             │
│ • Configuration files (YAML, JSON)                               │
│ • Metadata form definitions                                      │
│ • Glossary terms                                                 │
│ • Project documentation (README.md)                              │
│ • Infrastructure as Code (CloudFormation/Terraform)              │
│ • Dashboard definitions (QuickSight, Tableau)                    │
│ • Data quality rules                                             │
│ • Test scripts                                                   │
└──────────────────────────────────────────────────────────────────┘

STOCKÉS DANS S3 (Artifacts):
┌──────────────────────────────────────────────────────────────────┐
│ • Compiled JARs (pour Glue/EMR jobs)                             │
│ • Docker images (si containerized)                               │
│ • Lambda deployment packages                                     │
│ • Libraries/dependencies                                         │
│ • Sample data files (pour tests)                                 │
└──────────────────────────────────────────────────────────────────┘

CRÉÉS VIA API/CONSOLE (Infrastructure):
┌──────────────────────────────────────────────────────────────────┐
│ • SMUS Project (via DataZone API)                                │
│ • Domain Units                                                   │
│ • Project memberships                                            │
│ • Lake Formation permissions                                     │
│ • Glue Jobs, Crawlers                                           │
│ • Athena Named Queries                                          │
│ • EventBridge Rules                                             │
│ • CloudWatch Dashboards & Alarms                                │
└──────────────────────────────────────────────────────────────────┘

DONNÉES (Pas promues, mais partagées):
┌──────────────────────────────────────────────────────────────────┐
│ • Source data reste dans PROD account                            │
│ • PROD-DEV et PROD-STAGING accèdent via Lake Formation          │
│ • Pas de copie de données entre comptes                         │
│ • Seulement les permissions qui changent                         │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. ROLLBACK STRATEGY

```
┌────────────────────────────────────────────────────────────────────┐
│  PLAN DE ROLLBACK                                                  │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Scénario 1: Problème détecté pendant le déploiement              │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ • STOP le déploiement immédiatement                          │ │
│  │ • Revert to previous version (v1)                            │ │
│  │ • Update pointer: v2 → v1                                    │ │
│  │ • Notify users (service restored)                            │ │
│  │ • Investigation post-incident                                │ │
│  │ • Durée: 5-10 minutes                                        │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Scénario 2: Problème détecté après déploiement complet           │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ Si < 24h:                                                    │ │
│  │ • v1 encore disponible                                       │ │
│  │ • Switch back to v1 (5 minutes)                              │ │
│  │                                                               │ │
│  │ Si > 24h:                                                    │ │
│  │ • Hotfix deployment                                          │ │
│  │ • ou Restore from Git (re-deploy v1)                        │ │
│  │ • Durée: 30-60 minutes                                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  Conditions de Rollback:                                           │
│  • Error rate > 5%                                                │
│  • Performance degradation > 50%                                  │
│  • Critical bug discovered                                        │
│  • Security issue identified                                      │
│  • Business stakeholder request                                   │
└────────────────────────────────────────────────────────────────────┘
```

---

## 5. GOUVERNANCE & CONTRÔLES

```
┌────────────────────────────────────────────────────────────────────┐
│  QUI PEUT FAIRE QUOI ?                                             │
└────────────────────────────────────────────────────────────────────┘

PROD-DEV → PROD-STAGING:
┌──────────────────────────────────────────────────────────────────┐
│ Trigger Promotion:                                                │
│ • Data Engineers (via Git MR)                                    │
│ • Senior Data Engineers (approval)                               │
│                                                                   │
│ Approvals Required:                                              │
│ • 1x Senior Data Engineer (code review)                          │
│ • Automated tests must pass                                      │
│                                                                   │
│ Automation Level: 90%                                            │
│ • CI/CD fait tout automatiquement après MR approval              │
└──────────────────────────────────────────────────────────────────┘

PROD-STAGING → PROD:
┌──────────────────────────────────────────────────────────────────┐
│ Trigger Promotion:                                                │
│ • Product Owner (après UAT success)                              │
│ • Data Platform Manager                                          │
│                                                                   │
│ Approvals Required:                                              │
│ • Product Owner ✓                                                │
│ • Data Platform Manager ✓                                        │
│ • IT Operations Manager ✓                                        │
│ • CAB (si high-risk change) ✓                                   │
│                                                                   │
│ Automation Level: 70%                                            │
│ • CI/CD déploie après approbations m





## Ressources

- https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/concepts.html
- https://github.com/aws-solutions-library-samples/guidance-for-collaborative-unified-data-and-ai-development-on-aws
- https://aws.amazon.com/fr/blogs/big-data/cross-account-data-collaboration-with-amazon-datazone-and-aws-analytical-tools/
- https://dev.to/aws-builders/data-governance-on-aws-using-datazone-4li2
- https://aws.amazon.com/fr/blogs/big-data/foundational-blocks-of-amazon-sagemaker-unified-studio-an-admins-guide-to-implement-unified-access-to-all-your-data-analytics-and-ai/
- https://github.com/aws-solutions-library-samples/guidance-for-collaborative-unified-data-and-ai-development-on-aws
- https://docs.aws.amazon.com/sagemaker-unified-studio/latest/adminguide/account-pools.html
- https://www.ailearnhub.org/resources/inside-amazon-sagemaker-unified-studio-a-unified-data-analytics
- https://noise.getoto.net/2025/02/14/foundational-blocks-of-amazon-sagemaker-unified-studio-an-admins-guide-to-implement-unified-access-to-all-your-data-analytics-and-ai/
