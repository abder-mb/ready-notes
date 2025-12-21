
# SMUS Management SDK Reference

Complete Python SDK Guide for Amazon SageMaker Unified Studio

---

## 📚 SDK Color Legend

- 🟩 **sagemaker-studio** – SMUS Native SDK  
- 🟦 **boto3.datazone** – DataZone Backend  
- 🟨 **boto3.glue** – Glue Catalog  
- 🟥 **boto3.lakeformation** – Permissions  
- 🟪 **General boto3** – AWS Services  

---

## Feature Matrix

| Feature Area | SDK / Library | Key Functions | Documentation Links | Notes & Examples |
|-------------|---------------|---------------|---------------------|------------------|

### 🏢 DOMAIN MANAGEMENT

#### Domain Access & Info

- **SDK**: `sagemaker-studio`  
  - Install:

    ```
    pip install sagemaker-studio
    ```

- **Key functions**:
  - `Domain()` – Get domain info  
  - `domain.id` – Domain ID  
  - `domain.name` – Domain name  
  - `domain.region` – AWS Region  

- **Docs**:
  - [PyPI: sagemaker-studio](https://pypi.org/project/sagemaker-studio/)  
  - [SMUS Python Library Guide](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/python-library.html)  

- **Example**:

  ```
  from sagemaker_studio import Domain
  dom = Domain()
  print(dom.id)
  ```

---

### 📁 PROJECT MANAGEMENT

#### Project Operations

- **SDK**: `sagemaker-studio`  

- **Key functions**:
  - `Project()` – Get current project  
  - `project.id` – Project ID  
  - `project.name` – Project name  
  - `project.domain_id` – Domain ID  

- **Docs**:
  - [PyPI Documentation](https://pypi.org/project/sagemaker-studio/)

- **Example**:

  ```
  from sagemaker_studio import Project
  proj = Project()
  print(proj.name)
  ```

#### Create/Delete Projects

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_project()`  
  - `delete_project()`  
  - `update_project()`  
  - `list_projects()`  

- **Docs**:
  - [Boto3 DataZone API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html)

- **Notes**:
  - ⚠️ SMUS uses DataZone backend for project CRUD operations.

---

### 📊 CATALOG & ASSET MANAGEMENT

#### Search & Discover Assets

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `search()` – Search catalog  
  - `search_listings()` – Search published assets  
  - `get_asset()` – Get asset details  
  - `list_assets()` – List assets  

- **Docs**:
  - [DataZone Search API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/search.html)  
  - [DataZone API Quickstart](https://docs.aws.amazon.com/datazone/latest/userguide/quickstart-apis.html)

- **Example**:

  ```
  dz = boto3.client("datazone")
  results = dz.search(domainIdentifier="dzd_xxx", searchText="sales")
  ```

#### Asset Creation & Publishing

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_asset()`  
  - `create_asset_type()`  
  - `create_listing()`  
  - `update_asset()`  
  - `delete_asset()`  

- **Docs**:
  - [Create Asset API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_asset.html)

- **Notes**:
  - 💡 Assets must be created in a project context.

#### Glue Catalog Tables

- **SDK**: `boto3.glue`  

- **Key functions**:
  - `get_table()`  
  - `get_tables()`  
  - `get_databases()`  
  - `get_partitions()`  

- **Docs**:
  - [Boto3 Glue API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html)

- **Notes**:
  - For technical metadata (schemas, partitions).

---

### 🔐 SUBSCRIPTIONS & ACCESS REQUESTS

#### Subscription Requests

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_subscription_request()`  
  - `list_subscription_requests()`  
  - `get_subscription_request_details()`  
  - `accept_subscription_request()`  
  - `reject_subscription_request()`  
  - `cancel_subscription()`  

- **Docs**:
  - [Subscription Request API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_subscription_request.html)  
  - [Subscription Guide](https://docs.aws.amazon.com/datazone/latest/userguide/subscribe-to-data-assets-managed-by-datazone.html)

- **Example**:

  ```
  dz.create_subscription_request(
      domainIdentifier="dzd_xxx",
      subscribedListings=[...],
  )
  ```

#### List & Manage Subscriptions

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `list_subscriptions()`  
  - `get_subscription()`  
  - `revoke_subscription()`  

- **Docs**:
  - [List Subscriptions API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/list_subscriptions.html)

- **Notes**:
  - Filter by status: `APPROVED`, `REVOKED`, `CANCELLED`.

---

### 🔒 PERMISSIONS & ACCESS CONTROL

#### Lake Formation Permissions

- **SDK**: `boto3.lakeformation`  

- **Key functions**:
  - `grant_permissions()`  
  - `revoke_permissions()`  
  - `list_permissions()`  
  - `batch_grant_permissions()`  

- **Docs**:
  - [Lake Formation API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lakeformation.html)

- **Notes**:
  - ⚠️ Used for fine-grained access control on tables/columns.

#### Resource Shares (Cross-Account)

- **SDK**: `boto3.lakeformation`  

- **Key functions**:
  - `create_lake_formation_opt_in()`  
  - `list_lake_formation_opt_ins()`  
  - `get_resource_lf_tags()`  

- **Docs**:
  - [Lake Formation Guide](https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html)

- **Notes**:
  - For cross-account catalog sharing (Producer → Main Domain).

#### DataZone Environment Roles

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_project_membership()`  
  - `delete_project_membership()`  
  - `list_project_memberships()`  

- **Docs**:
  - [Project Membership API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_project_membership.html)

- **Notes**:
  - Manage who has access to projects.

---

### 🔌 CONNECTIONS & DATA SOURCES

#### Project Connections

- **SDK**: `sagemaker-studio`  

- **Key functions**:
  - `project.connection(name)`  
  - `project.list_connections()`  
  - `connection.create_client(service)`  
  - `connection.environment_id`  

- **Docs**:
  - [Connection Clients Guide](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/connection-clients.html)

- **Example**:

  ```
  athena_conn = project.connection("project.athena")
  env_id = athena_conn.environment_id
  ```

#### Create Connections

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_connection()`  
  - `delete_connection()`  
  - `get_connection()`  
  - `list_connections()`  

- **Docs**:
  - [DataZone Connections](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html)

- **Notes**:
  - Connect to Athena, Redshift, EMR, Glue, etc.

---

### 📂 DATA ACCESS & QUERYING

#### SQL Queries (Athena)

- **SDK**: `sagemaker-studio`  

- **Key functions**:
  - `sql.execute(query)`  
  - `sql.execute(query, connection)`  
  - `sql.to_dataframe(result)`  

- **Docs**:
  - [sagemaker-studio SQL Utils](https://pypi.org/project/sagemaker-studio/)

- **Example**:

  ```
  from sagemaker_studio import sql
  result = sql.execute("SELECT * FROM db.table")
  ```

#### DataFrame Operations (awswrangler)

- **SDK**: `awswrangler`  

- Install:

  ```
  pip install awswrangler
  ```

- **Key functions**:
  - `wr.athena.read_sql_query()`  
  - `wr.catalog.databases()`  
  - `wr.catalog.tables()`  
  - `wr.s3.read_parquet()`  

- **Docs**:
  - [AWS SDK for pandas Docs](https://aws-sdk-pandas.readthedocs.io/)

- **Notes**:
  - 💡 Best for working with pandas DataFrames in SMUS.

#### Spark Sessions

- **SDK**: `sagemaker-studio`  

- **Key functions**:
  - `spark.create_session()`  
  - `spark.execute(code)`  
  - `spark.get_dataframe()`  

- **Docs**:
  - [Data Engineering Sessions](https://pypi.org/project/sagemaker-studio-dataengineering-sessions/)

- **Notes**:
  - For Glue Interactive Sessions, EMR.

---

### 🔗 METADATA & LINEAGE

#### Asset Metadata Forms

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_form_type()`  
  - `get_form_type()`  
  - `list_metadata_generation_runs()`  

- **Docs**:
  - [DataZone Metadata API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html)

#### Data Lineage

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `get_lineage_node()`  
  - `list_lineage_node_history()`  
  - `post_lineage_event()`  

- **Docs**:
  - [Lineage API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/get_lineage_node.html)

- **Notes**:
  - Track upstream/downstream relationships.

#### Business Glossary

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_glossary()`  
  - `create_glossary_term()`  
  - `list_glossary_terms()`  
  - `update_glossary_term()`  

- **Docs**:
  - [Glossary API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_glossary.html)

- **Notes**:
  - Define business terms and definitions.

---

### ✅ DATA QUALITY & MONITORING

#### Data Quality Rules

- **SDK**: `boto3.glue`  

- **Key functions**:
  - `create_data_quality_ruleset()`  
  - `start_data_quality_rule_recommendation_run()`  
  - `get_data_quality_result()`  

- **Docs**:
  - [Glue Data Quality API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue/client/create_data_quality_ruleset.html)

- **Notes**:
  - Define and monitor quality rules.

#### Quality Runs & Results

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `list_metadata_generation_runs()`  
  - `start_metadata_generation_run()`  

- **Docs**:
  - [DataZone Metadata Gen](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html)

- **Notes**:
  - Automated metadata extraction.

---

### 🌍 ENVIRONMENTS & PROFILES

#### Environment Management

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_environment()`  
  - `delete_environment()`  
  - `get_environment()`  
  - `list_environments()`  
  - `update_environment()`  

- **Docs**:
  - [Environment API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_environment.html)

- **Notes**:
  - 💡 Environments = Compute contexts (Athena, Glue, EMR, Redshift).

#### Environment Profiles

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_environment_profile()`  
  - `get_environment_profile()`  
  - `list_environment_profiles()`  

- **Docs**:
  - [Environment Profile API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_environment_profile.html)

- **Notes**:
  - Templates for creating environments.

---

### 📓 NOTEBOOKS & IDE

#### Notebook Management

- **SDK**: Not available (UI only).  

- **Notes**:
  - No direct SDK for notebook CRUD operations.
  - ⚠️ Notebooks managed through SMUS UI only.  
    Use `sagemaker-studio` library **inside** notebooks.

- **Docs**:
  - [Notebooks User Guide](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/notebooks.html)

#### IDE Spaces

- **SDK**: Not available (UI only).  

- **Notes**:
  - No direct SDK for IDE space management.
  - ⚠️ JupyterLab and Code Editor spaces managed via UI.

- **Docs**:
  - [IDE Spaces Guide](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/ide-spaces.html)

---

### ⚙️ WORKFLOWS & AUTOMATION

#### Notifications & Events

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `list_notifications()`  
  - `get_notification()`  
  - `create_subscription_target()`  

- **Docs**:
  - [Notifications API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/list_notifications.html)

- **Notes**:
  - Track subscription approvals, data changes.

#### Task Automation

- **SDK**: `boto3` (EventBridge)  

- **Key functions**:
  - `put_rule()`  
  - `put_targets()`  
  - `put_events()`  

- **Docs**:
  - [EventBridge API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html)

- **Notes**:
  - Trigger Lambda functions on SMUS events.

---

### 👨‍💼 ADMINISTRATION

#### User Groups & Members

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `create_group_profile()`  
  - `get_group_profile()`  
  - `search_group_profiles()`  
  - `search_user_profiles()`  

- **Docs**:
  - [Group Profile API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_group_profile.html)

- **Notes**:
  - Manage domain users and groups.

#### Domain Settings

- **SDK**: `boto3.datazone`  

- **Key functions**:
  - `get_domain()`  
  - `update_domain()`  
  - `list_domains()`  

- **Docs**:
  - [Domain API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/get_domain.html)

- **Notes**:
  - View and update domain configuration.

---

### 🔧 UTILITY & HELPER FUNCTIONS

#### Authentication & Context

- **SDK**: `sagemaker-studio`  

- **Key functions**:
  - `ClientConfig()`  
  - `get_caller_identity()`  
  - `get_execution_role()`  

- **Docs**:
  - [SMUS Python Library](https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/python-library.html)

- **Example**:

  ```
  from sagemaker_studio import ClientConfig
  config = ClientConfig()
  ```

#### Presigned URLs & File Access

- **SDK**: `boto3.s3`  

- **Key functions**:
  - `generate_presigned_url()`  
  - `upload_file()`  
  - `download_file()`  

- **Docs**:
  - [S3 API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)

- **Notes**:
  - Access data stored in S3.

---

## 📦 Installation Commands

```
# Core SMUS Library
pip install sagemaker-studio

# DataZone API (included in boto3)
pip install boto3 --upgrade

# Data Wrangling
pip install awswrangler

# Pandas Integration
pip install pandas pyarrow
```

---

## ⚠️ Important Notes

- **DataZone is the Backend**: SMUS uses AWS DataZone as its underlying service for catalog, projects, and subscriptions.  
- **sagemaker-studio library**: Use this *inside* SMUS notebooks/IDE for context-aware operations.  
- **boto3.datazone**: Use this for programmatic management from *outside* SMUS.  
- **Domain ID Required**: Most DataZone APIs require the domain identifier (`dzd_xxxxx`).  
- **Project Context**: Many operations must be performed within a project context.  
- **No Direct Notebook API**: Notebook management is UI-only; use the library inside notebooks.  
- **Permissions**: Ensure your IAM role has DataZone permissions for API operations.

---

## 🔗 Essential Documentation Links

### SMUS Documentation

- [User Guide](https://docs.aws.amazon.com/sagemaker-unified-studio/)  
- [Python Library](https://pypi.org/project/sagemaker-studio/)

### DataZone API

- [Boto3 Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html)  
- [DataZone Guide](https://docs.aws.amazon.com/datazone/)

### Glue & Lake Formation

- [Glue API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html)  
- [Lake Formation API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lakeformation.html)

### Data Tools

- [AWS SDK for pandas](https://aws-sdk-pandas.readthedocs.io/)  
- [Athena API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/athena.html)



---------------------

## html version

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SMUS SDK Reference</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            padding: 20px;
            min-height: 100vh;
        }
        
        .container {
            max-width: 1600px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 25px 80px rgba(0,0,0,0.4);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            padding: 40px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.8em;
            margin-bottom: 15px;
            font-weight: 700;
        }
        
        .header p {
            font-size: 1.2em;
            opacity: 0.95;
        }
        
        .legend {
            padding: 30px 40px;
            background: #f8f9fa;
            border-bottom: 3px solid #e9ecef;
        }
        
        .legend h3 {
            margin-bottom: 15px;
            color: #1e3c72;
            font-size: 1.3em;
        }
        
        .legend-items {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
        }
        
        .legend-item {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        
        .legend-color {
            width: 20px;
            height: 20px;
            border-radius: 4px;
        }
        
        .table-wrapper {
            overflow-x: auto;
            padding: 20px 40px 40px 40px;
        }
        
        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 14px;
        }
        
        thead {
            position: sticky;
            top: 0;
            z-index: 10;
        }
        
        th {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            padding: 18px 15px;
            text-align: left;
            font-weight: 600;
            font-size: 15px;
            border-right: 1px solid rgba(255,255,255,0.1);
        }
        
        th:last-child {
            border-right: none;
        }
        
        tbody tr {
            border-bottom: 1px solid #e9ecef;
            transition: all 0.2s;
        }
        
        tbody tr:hover {
            background: #f8f9fa;
            transform: scale(1.001);
            box-shadow: 0 2px 8px rgba(0,0,0,0.08);
        }
        
        td {
            padding: 16px 15px;
            vertical-align: top;
            border-right: 1px solid #f0f0f0;
        }
        
        td:last-child {
            border-right: none;
        }
        
        .feature-name {
            font-weight: 600;
            color: #1e3c72;
            font-size: 15px;
        }
        
        .sdk-name {
            font-weight: 600;
            padding: 6px 12px;
            border-radius: 6px;
            display: inline-block;
            margin-bottom: 5px;
        }
        
        .sdk-sagemaker {
            background: #d4edda;
            color: #155724;
        }
        
        .sdk-datazone {
            background: #d1ecf1;
            color: #0c5460;
        }
        
        .sdk-glue {
            background: #fff3cd;
            color: #856404;
        }
        
        .sdk-lakeformation {
            background: #f8d7da;
            color: #721c24;
        }
        
        .sdk-boto3 {
            background: #e7e7ff;
            color: #3333cc;
        }
        
        .not-available {
            color: #dc3545;
            font-weight: 600;
            font-style: italic;
        }
        
        .workaround {
            color: #fd7e14;
            font-weight: 600;
        }
        
        .function-list {
            margin: 8px 0;
            padding-left: 0;
            list-style: none;
        }
        
        .function-list li {
            padding: 5px 0;
            padding-left: 20px;
            position: relative;
        }
        
        .function-list li:before {
            content: "▸";
            position: absolute;
            left: 0;
            color: #2a5298;
            font-weight: bold;
        }
        
        .code {
            background: #f4f4f4;
            padding: 3px 8px;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            font-size: 13px;
            color: #e83e8c;
        }
        
        .link {
            color: #007bff;
            text-decoration: none;
            font-weight: 500;
            display: inline-block;
            margin: 3px 0;
        }
        
        .link:hover {
            text-decoration: underline;
            color: #0056b3;
        }
        
        .notes {
            margin-top: 8px;
            padding: 10px;
            background: #fff3cd;
            border-left: 4px solid #ffc107;
            border-radius: 4px;
            font-size: 13px;
            color: #856404;
        }
        
        .category-header {
            background: #e9ecef !important;
            font-weight: 700;
            font-size: 16px;
            color: #495057 !important;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        
        .install-cmd {
            background: #2d2d2d;
            color: #00ff00;
            padding: 10px 15px;
            border-radius: 6px;
            font-family: 'Courier New', monospace;
            margin: 5px 0;
            font-size: 13px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🔧 SMUS Management SDK Reference</h1>
            <p>Complete Python SDK Guide for Amazon SageMaker Unified Studio</p>
        </div>
        
        <div class="legend">
            <h3>📚 SDK Color Legend</h3>
            <div class="legend-items">
                <div class="legend-item">
                    <div class="legend-color" style="background: #d4edda;"></div>
                    <span><strong>sagemaker-studio</strong> - SMUS Native SDK</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #d1ecf1;"></div>
                    <span><strong>boto3.datazone</strong> - DataZone Backend</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #fff3cd;"></div>
                    <span><strong>boto3.glue</strong> - Glue Catalog</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #f8d7da;"></div>
                    <span><strong>boto3.lakeformation</strong> - Permissions</span>
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background: #e7e7ff;"></div>
                    <span><strong>General boto3</strong> - AWS Services</span>
                </div>
            </div>
        </div>
        
        <div class="table-wrapper">
            <table>
                <thead>
                    <tr>
                        <td>
                            Custom forms for business metadata
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Data Lineage</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">get_lineage_node()</span></li>
                                <li><span class="code">list_lineage_node_history()</span></li>
                                <li><span class="code">post_lineage_event()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/get_lineage_node.html" class="link" target="_blank">Lineage API</a>
                        </td>
                        <td>
                            Track upstream/downstream relationships
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Business Glossary</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_glossary()</span></li>
                                <li><span class="code">create_glossary_term()</span></li>
                                <li><span class="code">list_glossary_terms()</span></li>
                                <li><span class="code">update_glossary_term()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_glossary.html" class="link" target="_blank">Glossary API</a>
                        </td>
                        <td>
                            Define business terms and definitions
                        </td>
                    </tr>
                    
                    <!-- DATA QUALITY -->
                    <tr>
                        <td colspan="5" class="category-header">✅ DATA QUALITY & MONITORING</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Data Quality Rules</td>
                        <td>
                            <span class="sdk-name sdk-glue">boto3.glue</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_data_quality_ruleset()</span></li>
                                <li><span class="code">start_data_quality_rule_recommendation_run()</span></li>
                                <li><span class="code">get_data_quality_result()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue/client/create_data_quality_ruleset.html" class="link" target="_blank">Glue Data Quality API</a>
                        </td>
                        <td>
                            Define and monitor quality rules
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Quality Runs & Results</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">list_metadata_generation_runs()</span></li>
                                <li><span class="code">start_metadata_generation_run()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html" class="link" target="_blank">DataZone Metadata Gen</a>
                        </td>
                        <td>
                            Automated metadata extraction
                        </td>
                    </tr>
                    
                    <!-- ENVIRONMENTS -->
                    <tr>
                        <td colspan="5" class="category-header">🌍 ENVIRONMENTS & PROFILES</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Environment Management</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_environment()</span></li>
                                <li><span class="code">delete_environment()</span></li>
                                <li><span class="code">get_environment()</span></li>
                                <li><span class="code">list_environments()</span></li>
                                <li><span class="code">update_environment()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_environment.html" class="link" target="_blank">Environment API</a>
                        </td>
                        <td>
                            <div class="notes">💡 Environments = Compute contexts (Athena, Glue, EMR, Redshift)</div>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Environment Profiles</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_environment_profile()</span></li>
                                <li><span class="code">get_environment_profile()</span></li>
                                <li><span class="code">list_environment_profiles()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_environment_profile.html" class="link" target="_blank">Environment Profile API</a>
                        </td>
                        <td>
                            Templates for creating environments
                        </td>
                    </tr>
                    
                    <!-- NOTEBOOKS -->
                    <tr>
                        <td colspan="5" class="category-header">📓 NOTEBOOKS & IDE</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Notebook Management</td>
                        <td>
                            <span class="not-available">Not Available</span><br>
                            <span class="workaround">UI Only</span>
                        </td>
                        <td>
                            No direct SDK for notebook CRUD operations
                        </td>
                        <td>
                            <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/notebooks.html" class="link" target="_blank">Notebooks User Guide</a>
                        </td>
                        <td>
                            <div class="notes">⚠️ Notebooks managed through SMUS UI only. Use sagemaker-studio library INSIDE notebooks</div>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">IDE Spaces</td>
                        <td>
                            <span class="not-available">Not Available</span><br>
                            <span class="workaround">UI Only</span>
                        </td>
                        <td>
                            No direct SDK for IDE space management
                        </td>
                        <td>
                            <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/ide-spaces.html" class="link" target="_blank">IDE Spaces Guide</a>
                        </td>
                        <td>
                            <div class="notes">⚠️ JupyterLab and Code Editor spaces managed via UI</div>
                        </td>
                    </tr>
                    
                    <!-- WORKFLOWS -->
                    <tr>
                        <td colspan="5" class="category-header">⚙️ WORKFLOWS & AUTOMATION</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Notifications & Events</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">list_notifications()</span></li>
                                <li><span class="code">get_notification()</span></li>
                                <li><span class="code">create_subscription_target()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/list_notifications.html" class="link" target="_blank">Notifications API</a>
                        </td>
                        <td>
                            Track subscription approvals, data changes
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Task Automation</td>
                        <td>
                            <span class="sdk-name sdk-boto3">boto3 (EventBridge)</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">put_rule()</span></li>
                                <li><span class="code">put_targets()</span></li>
                                <li><span class="code">put_events()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/events.html" class="link" target="_blank">EventBridge API</a>
                        </td>
                        <td>
                            Trigger Lambda functions on SMUS events
                        </td>
                    </tr>
                    
                    <!-- ADMIN -->
                    <tr>
                        <td colspan="5" class="category-header">👨‍💼 ADMINISTRATION</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">User Groups & Members</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_group_profile()</span></li>
                                <li><span class="code">get_group_profile()</span></li>
                                <li><span class="code">search_group_profiles()</span></li>
                                <li><span class="code">search_user_profiles()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_group_profile.html" class="link" target="_blank">Group Profile API</a>
                        </td>
                        <td>
                            Manage domain users and groups
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Domain Settings</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">get_domain()</span></li>
                                <li><span class="code">update_domain()</span></li>
                                <li><span class="code">list_domains()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/get_domain.html" class="link" target="_blank">Domain API</a>
                        </td>
                        <td>
                            View and update domain configuration
                        </td>
                    </tr>
                    
                    <!-- ADDITIONAL -->
                    <tr>
                        <td colspan="5" class="category-header">🔧 UTILITY & HELPER FUNCTIONS</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Authentication & Context</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">ClientConfig()</span></li>
                                <li><span class="code">get_caller_identity()</span></li>
                                <li><span class="code">get_execution_role()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/python-library.html" class="link" target="_blank">SMUS Python Library</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">from sagemaker_studio import ClientConfig</span><br>
                            <span class="code">config = ClientConfig()</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Presigned URLs & File Access</td>
                        <td>
                            <span class="sdk-name sdk-boto3">boto3.s3</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">generate_presigned_url()</span></li>
                                <li><span class="code">upload_file()</span></li>
                                <li><span class="code">download_file()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html" class="link" target="_blank">S3 API</a>
                        </td>
                        <td>
                            Access data stored in S3
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        
        <div style="padding: 40px; background: #f8f9fa; border-top: 3px solid #e9ecef;">
            <h2 style="color: #1e3c72; margin-bottom: 20px;">📦 Installation Commands</h2>
            <div style="background: #2d2d2d; color: #00ff00; padding: 20px; border-radius: 10px; font-family: 'Courier New', monospace;">
                # Core SMUS Library<br>
                pip install sagemaker-studio<br><br>
                
                # DataZone API (included in boto3)<br>
                pip install boto3 --upgrade<br><br>
                
                # Data Wrangling<br>
                pip install awswrangler<br><br>
                
                # Pandas Integration<br>
                pip install pandas pyarrow
            </div>
            
            <h2 style="color: #1e3c72; margin-top: 30px; margin-bottom: 15px;">⚠️ Important Notes</h2>
            <div style="background: white; padding: 20px; border-radius: 10px; border-left: 5px solid #ffc107;">
                <ul style="padding-left: 20px; line-height: 1.8;">
                    <li><strong>DataZone is the Backend:</strong> SMUS uses AWS DataZone as its underlying service for catalog, projects, and subscriptions</li>
                    <li><strong>sagemaker-studio library:</strong> Use this INSIDE SMUS notebooks/IDE for context-aware operations</li>
                    <li><strong>boto3.datazone:</strong> Use this for programmatic management FROM OUTSIDE SMUS</li>
                    <li><strong>Domain ID Required:</strong> Most DataZone APIs require the domain identifier (dzd_xxxxx)</li>
                    <li><strong>Project Context:</strong> Many operations must be performed within a project context</li>
                    <li><strong>No Direct Notebook API:</strong> Notebook management is UI-only; use the library inside notebooks</li>
                    <li><strong>Permissions:</strong> Ensure your IAM role has DataZone permissions for API operations</li>
                </ul>
            </div>
            
            <h2 style="color: #1e3c72; margin-top: 30px; margin-bottom: 15px;">🔗 Essential Documentation Links</h2>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px;">
                <div style="background: white; padding: 15px; border-radius: 8px;">
                    <h4 style="color: #2a5298; margin-bottom: 10px;">SMUS Documentation</h4>
                    <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/" class="link" target="_blank">User Guide</a><br>
                    <a href="https://pypi.org/project/sagemaker-studio/" class="link" target="_blank">Python Library</a>
                </div>
                <div style="background: white; padding: 15px; border-radius: 8px;">
                    <h4 style="color: #2a5298; margin-bottom: 10px;">DataZone API</h4>
                    <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html" class="link" target="_blank">Boto3 Reference</a><br>
                    <a href="https://docs.aws.amazon.com/datazone/" class="link" target="_blank">DataZone Guide</a>
                </div>
                <div style="background: white; padding: 15px; border-radius: 8px;">
                    <h4 style="color: #2a5298; margin-bottom: 10px;">Glue & Lake Formation</h4>
                    <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html" class="link" target="_blank">Glue API</a><br>
                    <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lakeformation.html" class="link" target="_blank">Lake Formation API</a>
                </div>
                <div style="background: white; padding: 15px; border-radius: 8px;">
                    <h4 style="color: #2a5298; margin-bottom: 10px;">Data Tools</h4>
                    <a href="https://aws-sdk-pandas.readthedocs.io/" class="link" target="_blank">AWS SDK for pandas</a><br>
                    <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/athena.html" class="link" target="_blank">Athena API</a>
                </div>
            </div>
        </div>
    </div>
</body>
</html>h style="width: 12%;">Feature Area</th>
                        <th style="width: 15%;">SDK / Library</th>
                        <th style="width: 25%;">Key Functions</th>
                        <th style="width: 23%;">Documentation Links</th>
                        <th style="width: 25%;">Notes & Examples</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- DOMAIN MANAGEMENT -->
                    <tr>
                        <td colspan="5" class="category-header">🏢 DOMAIN MANAGEMENT</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Domain Access & Info</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                            <div class="install-cmd">pip install sagemaker-studio</div>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">Domain()</span> - Get domain info</li>
                                <li><span class="code">domain.id</span> - Domain ID</li>
                                <li><span class="code">domain.name</span> - Domain name</li>
                                <li><span class="code">domain.region</span> - AWS Region</li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://pypi.org/project/sagemaker-studio/" class="link" target="_blank">PyPI: sagemaker-studio</a><br>
                            <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/python-library.html" class="link" target="_blank">SMUS Python Library Guide</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">from sagemaker_studio import Domain</span><br>
                            <span class="code">dom = Domain()</span><br>
                            <span class="code">print(dom.id)</span>
                        </td>
                    </tr>
                    
                    <!-- PROJECT MANAGEMENT -->
                    <tr>
                        <td colspan="5" class="category-header">📁 PROJECT MANAGEMENT</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Project Operations</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">Project()</span> - Get current project</li>
                                <li><span class="code">project.id</span> - Project ID</li>
                                <li><span class="code">project.name</span> - Project name</li>
                                <li><span class="code">project.domain_id</span> - Domain ID</li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://pypi.org/project/sagemaker-studio/" class="link" target="_blank">PyPI Documentation</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">from sagemaker_studio import Project</span><br>
                            <span class="code">proj = Project()</span><br>
                            <span class="code">print(proj.name)</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Create/Delete Projects</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_project()</span></li>
                                <li><span class="code">delete_project()</span></li>
                                <li><span class="code">update_project()</span></li>
                                <li><span class="code">list_projects()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html" class="link" target="_blank">Boto3 DataZone API</a>
                        </td>
                        <td>
                            <div class="notes">⚠️ SMUS uses DataZone backend for project CRUD operations</div>
                        </td>
                    </tr>
                    
                    <!-- CATALOG MANAGEMENT -->
                    <tr>
                        <td colspan="5" class="category-header">📊 CATALOG & ASSET MANAGEMENT</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Search & Discover Assets</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">search()</span> - Search catalog</li>
                                <li><span class="code">search_listings()</span> - Search published assets</li>
                                <li><span class="code">get_asset()</span> - Get asset details</li>
                                <li><span class="code">list_assets()</span> - List assets</li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/search.html" class="link" target="_blank">DataZone Search API</a><br>
                            <a href="https://docs.aws.amazon.com/datazone/latest/userguide/quickstart-apis.html" class="link" target="_blank">DataZone API Quickstart</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">dz = boto3.client('datazone')</span><br>
                            <span class="code">results = dz.search(domainIdentifier='dzd_xxx', searchText='sales')</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Asset Creation & Publishing</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_asset()</span></li>
                                <li><span class="code">create_asset_type()</span></li>
                                <li><span class="code">create_listing()</span></li>
                                <li><span class="code">update_asset()</span></li>
                                <li><span class="code">delete_asset()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_asset.html" class="link" target="_blank">Create Asset API</a>
                        </td>
                        <td>
                            <div class="notes">💡 Assets must be created in a project context</div>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Glue Catalog Tables</td>
                        <td>
                            <span class="sdk-name sdk-glue">boto3.glue</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">get_table()</span></li>
                                <li><span class="code">get_tables()</span></li>
                                <li><span class="code">get_databases()</span></li>
                                <li><span class="code">get_partitions()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html" class="link" target="_blank">Boto3 Glue API</a>
                        </td>
                        <td>
                            <strong>For technical metadata</strong> (schemas, partitions)
                        </td>
                    </tr>
                    
                    <!-- SUBSCRIPTIONS & ACCESS -->
                    <tr>
                        <td colspan="5" class="category-header">🔐 SUBSCRIPTIONS & ACCESS REQUESTS</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Subscription Requests</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_subscription_request()</span></li>
                                <li><span class="code">list_subscription_requests()</span></li>
                                <li><span class="code">get_subscription_request_details()</span></li>
                                <li><span class="code">accept_subscription_request()</span></li>
                                <li><span class="code">reject_subscription_request()</span></li>
                                <li><span class="code">cancel_subscription()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_subscription_request.html" class="link" target="_blank">Subscription Request API</a><br>
                            <a href="https://docs.aws.amazon.com/datazone/latest/userguide/subscribe-to-data-assets-managed-by-datazone.html" class="link" target="_blank">Subscription Guide</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">dz.create_subscription_request(</span><br>
                            <span class="code">&nbsp;&nbsp;domainIdentifier='dzd_xxx',</span><br>
                            <span class="code">&nbsp;&nbsp;subscribedListings=[...]</span><br>
                            <span class="code">)</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">List & Manage Subscriptions</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">list_subscriptions()</span></li>
                                <li><span class="code">get_subscription()</span></li>
                                <li><span class="code">revoke_subscription()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/list_subscriptions.html" class="link" target="_blank">List Subscriptions API</a>
                        </td>
                        <td>
                            Filter by status: APPROVED, REVOKED, CANCELLED
                        </td>
                    </tr>
                    
                    <!-- PERMISSIONS -->
                    <tr>
                        <td colspan="5" class="category-header">🔒 PERMISSIONS & ACCESS CONTROL</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Lake Formation Permissions</td>
                        <td>
                            <span class="sdk-name sdk-lakeformation">boto3.lakeformation</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">grant_permissions()</span></li>
                                <li><span class="code">revoke_permissions()</span></li>
                                <li><span class="code">list_permissions()</span></li>
                                <li><span class="code">batch_grant_permissions()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lakeformation.html" class="link" target="_blank">Lake Formation API</a>
                        </td>
                        <td>
                            <div class="notes">⚠️ Used for fine-grained access control on tables/columns</div>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Resource Shares (Cross-Account)</td>
                        <td>
                            <span class="sdk-name sdk-lakeformation">boto3.lakeformation</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_lake_formation_opt_in()</span></li>
                                <li><span class="code">list_lake_formation_opt_ins()</span></li>
                                <li><span class="code">get_resource_lf_tags()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/what-is-lake-formation.html" class="link" target="_blank">Lake Formation Guide</a>
                        </td>
                        <td>
                            For cross-account catalog sharing (Producer → Main Domain)
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">DataZone Environment Roles</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_project_membership()</span></li>
                                <li><span class="code">delete_project_membership()</span></li>
                                <li><span class="code">list_project_memberships()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone/client/create_project_membership.html" class="link" target="_blank">Project Membership API</a>
                        </td>
                        <td>
                            Manage who has access to projects
                        </td>
                    </tr>
                    
                    <!-- CONNECTIONS -->
                    <tr>
                        <td colspan="5" class="category-header">🔌 CONNECTIONS & DATA SOURCES</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Project Connections</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">project.connection(name)</span></li>
                                <li><span class="code">project.list_connections()</span></li>
                                <li><span class="code">connection.create_client(service)</span></li>
                                <li><span class="code">connection.environment_id</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/connection-clients.html" class="link" target="_blank">Connection Clients Guide</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">athena_conn = project.connection('project.athena')</span><br>
                            <span class="code">env_id = athena_conn.environment_id</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Create Connections</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_connection()</span></li>
                                <li><span class="code">delete_connection()</span></li>
                                <li><span class="code">get_connection()</span></li>
                                <li><span class="code">list_connections()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html" class="link" target="_blank">DataZone Connections</a>
                        </td>
                        <td>
                            Connect to Athena, Redshift, EMR, Glue, etc.
                        </td>
                    </tr>
                    
                    <!-- DATA ACCESS -->
                    <tr>
                        <td colspan="5" class="category-header">📂 DATA ACCESS & QUERYING</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">SQL Queries (Athena)</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">sql.execute(query)</span></li>
                                <li><span class="code">sql.execute(query, connection)</span></li>
                                <li><span class="code">sql.to_dataframe(result)</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://pypi.org/project/sagemaker-studio/" class="link" target="_blank">sagemaker-studio SQL Utils</a>
                        </td>
                        <td>
                            <strong>Example:</strong><br>
                            <span class="code">from sagemaker_studio import sql</span><br>
                            <span class="code">result = sql.execute("SELECT * FROM db.table")</span>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">DataFrame Operations (awswrangler)</td>
                        <td>
                            <span class="sdk-name sdk-boto3">awswrangler</span>
                            <div class="install-cmd">pip install awswrangler</div>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">wr.athena.read_sql_query()</span></li>
                                <li><span class="code">wr.catalog.databases()</span></li>
                                <li><span class="code">wr.catalog.tables()</span></li>
                                <li><span class="code">wr.s3.read_parquet()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://aws-sdk-pandas.readthedocs.io/" class="link" target="_blank">AWS SDK for pandas Docs</a>
                        </td>
                        <td>
                            <div class="notes">💡 Best for working with pandas DataFrames in SMUS</div>
                        </td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Spark Sessions</td>
                        <td>
                            <span class="sdk-name sdk-sagemaker">sagemaker-studio</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">spark.create_session()</span></li>
                                <li><span class="code">spark.execute(code)</span></li>
                                <li><span class="code">spark.get_dataframe()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://pypi.org/project/sagemaker-studio-dataengineering-sessions/" class="link" target="_blank">Data Engineering Sessions</a>
                        </td>
                        <td>
                            For Glue Interactive Sessions, EMR
                        </td>
                    </tr>
                    
                    <!-- METADATA & LINEAGE -->
                    <tr>
                        <td colspan="5" class="category-header">🔗 METADATA & LINEAGE</td>
                    </tr>
                    
                    <tr>
                        <td class="feature-name">Asset Metadata Forms</td>
                        <td>
                            <span class="sdk-name sdk-datazone">boto3.datazone</span>
                        </td>
                        <td>
                            <ul class="function-list">
                                <li><span class="code">create_form_type()</span></li>
                                <li><span class="code">get_form_type()</span></li>
                                <li><span class="code">list_metadata_generation_runs()</span></li>
                            </ul>
                        </td>
                        <td>
                            <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html" class="link" target="_blank">DataZone Metadata API</a>
                        </td>
                        <t



I've created a comprehensive SDK reference table for managing SMUS (Amazon SageMaker Unified Studio). Here are the key findings:

## **Major Insights:**

### **1. SMUS Uses DataZone as Backend**
- **boto3 DataZone client** is the primary SDK for programmatic SMUS management
- Most catalog, project, and subscription operations go through DataZone APIs
- The domain identifier (dzd_xxxxx) is required for most operations

### **2. Three Main SDKs:**

**A. `sagemaker-studio` (Native SMUS Library)**
- Use INSIDE SMUS notebooks/IDE
- Context-aware (automatically knows domain, project)
- Best for: Domain info, project info, SQL execution, connections

**B. `boto3.client('datazone')` (Management API)**
- Use FROM OUTSIDE SMUS for automation
- Full CRUD operations
- Best for: Creating projects, managing subscriptions, asset publishing, permissions

**C. `boto3.client('glue')` + `boto3.client('lakeformation')`
- For technical metadata and permissions
- Glue: Schema, tables, databases
- Lake Formation: Cross-account sharing, fine-grained permissions

### **3. Key Limitations:**
- **No Notebook Management API** - UI only
- **No IDE Space API** - UI only
- Must use DataZone APIs for most administrative tasks

### **4. Common Operations:**

**Search Assets:**
```python
dz = boto3.client('datazone')
results = dz.search(
    domainIdentifier='dzd_xxxxx',
    searchText='sales data'
)
```

**Request Access:**
```python
dz.create_subscription_request(
    domainIdentifier='dzd_xxxxx',
    subscribedListings=[...]
)
```

**Query Data:**
```python
from sagemaker_studio import sql
result = sql.execute("SELECT * FROM sales_db.orders")
```

------------



NB-Studio:

## Managmement APP

The creation of a management application for Amazon SageMaker Unified Studio (SMUS) using AWS Python SDKs (like Boto3) would primarily involve interfacing with the specialized Amazon SageMaker Unified Studio Library for Python for foundational project interactions, and the underlying AWS DataZone and other AWS service SDKs for catalog, asset, and security management.
The sources explicitly mention the existence and capabilities of the sagemaker_studio Python library and provide many API and CLI references that map to AWS DataZone service actions.
Here is a table detailing the relevant management features, the appropriate SDKs, and corresponding functions as described in the provided sources:



The creation of a management application for Amazon SageMaker Unified Studio (SMUS) using AWS Python SDKs (like Boto3) would primarily involve interfacing with the specialized **Amazon SageMaker Unified Studio Library for Python** for foundational project interactions, and the underlying **AWS DataZone** and other AWS service SDKs for catalog, asset, and security management.

The sources explicitly mention the existence and capabilities of the `sagemaker_studio` Python library and provide many API and CLI references that map to AWS DataZone service actions.

Here is a table detailing the relevant management features, the appropriate SDKs, and corresponding functions as described in the provided sources:



| Feature/Functionality | AWS SDK/Library | Adapted Functions (API/Client Methods) | Link to SDK Documentation | Additional Context |
| :--- | :--- | :--- | :--- | :--- |
| **Project & Domain Information (Read)** | **SageMaker Unified Studio Library for Python** (`sagemaker_studio`) | `Domain(id=...)` (for domain details) `Project(name=..., domain_id=...)` (for project details) `proj.iam_role`, `proj.s3.root` (for role ARN and S3 paths) | Not Available (Source is internal guide documentation) | This library provides direct access to project and domain properties like ID, ARN, status, execution role ARN, and various S3 paths. It is supported in JupyterLab/Code Editor. |
| **Asset Creation (Data Sources)** | **AWS DataZone SDK** (inferred from CLI) | `CreateDataSource` (e.g., for AWS Glue, Amazon Redshift, Amazon SageMaker AI source types) | Not Available | Creating data sources adds metadata about external tables/views to the SMUS project inventory and enables publishing to the centralized Catalog. The command uses `aws datazone create-data-source`. |
| **Metadata Management (Data Quality)** | **AWS DataZone SDK** (inferred from CLI/API) | `PostTimeSeriesDataPoints` `ListTimeSeriesDataPoints` `DeleteTimeSeriesDataPoints` | Not Available | Used for importing data quality metrics from AWS Glue or third-party solutions for custom asset types. Requires the Domain ID and Entity ID (asset). |
| **Asset Metadata Export** | **AWS DataZone SDK** (inferred from CLI/API) | `PutDataExportConfiguration` | Not Available | Enables exporting catalog asset metadata into a queryable Apache Iceberg table in Amazon S3 for reporting and auditing purposes. |
| **Permissions (AWS Glue Access Grants)** | **AWS Lake Formation SDK** (inferred from service actions) | `GrantPermissions` (for managing DESCRIBE, SELECT permissions via Data Cell Filters) | Not Available | For managed AWS Glue tables, SMUS implements access control by creating grants in AWS Lake Formation, typically using Data Cell Filters when row/column filters are applied. |
| **Permissions (Redshift Access Grants)** | **Amazon Redshift SDK** (inferred from service actions) | Functions related to creating scoped-down late binding views and managing datashares | Not Available | For managed Amazon Redshift assets, access is materialized by creating late binding views and managing datashares between publisher and subscriber projects. |
| **External Data Query Authentication** | **AWS SSO Oauth** (inferred from API) | `RedeemAccessToken` | Not Available | Used by external analytics applications (via Athena JDBC driver) to exchange an Identity Center access token for `AmazonDataZoneDomainExecutionRole` credentials, enabling query access to governed data. |
| **Automation Workflow Integration** | **Amazon EventBridge SDK** | Actions related to event consumption, forwarding, and rules configuration | Not Available | Project creation emits a `CreateProject` event captured by AWS CloudTrail and bridged to an EventBridge bus. Subscription approval for unmanaged assets publishes an event to EventBridge, triggering custom fulfillment handlers. |
| **Notebook Execution (Headless)** | **SageMaker Unified Studio Library for Python** (`sagemaker_studio`) | `execution_client.start_execution` (local or remote) `execution_client.get_execution` `execution_client.list_executions` `execution_client.stop_execution` | Not Available (Source is internal guide documentation) | Allows notebooks to be executed headlessly (without an active UI session) either in the user's space (local) or on remote compute. |
