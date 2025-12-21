# SMUS SDK Reference - Part 2: Complete Examples & Advanced Patterns

## 📦 Complete Example 1: End-to-End Data Discovery to Analysis

```python
"""
Complete workflow: Discover data → Subscribe → Query → Process → Store
"""

import boto3
from sagemaker_studio import Domain, Project, sql
import awswrangler as wr
import pandas as pd

# ========================================
# 1. Get domain and project context
# ========================================
domain = Domain()
project = Project()
print(f"Working in domain: {domain.name}")
print(f"Working in project: {project.name}")
print(f"Domain ID: {domain.id}")
print(f"Project ID: {project.id}")

# ========================================
# 2. Search for datasets in SMUS catalog
# ========================================
dz = boto3.client('datazone')
search_results = dz.search(
    domainIdentifier=domain.id,
    searchText='customer data',
    searchScope='ASSET',
    maxResults=20
)

print(f"\nFound {len(search_results['items'])} datasets:")
for idx, item in enumerate(search_results['items']):
    asset = item['assetItem']
    print(f"{idx + 1}. {asset['name']} - {asset.get('description', 'No description')}")

# ========================================
# 3. Subscribe to a dataset
# ========================================
asset_id = search_results['items'][0]['assetItem']['identifier']
print(f"\nRequesting access to asset: {asset_id}")

subscription = dz.create_subscription_request(
    domainIdentifier=domain.id,
    subscribedListings=[{
        'identifier': asset_id
    }],
    subscribedPrincipals=[{
        'project': {'identifier': project.id}
    }],
    requestReason='ML model training - customer segmentation project'
)

print(f"Subscription request created: {subscription['id']}")
print(f"Status: {subscription['status']}")

# ========================================
# 4. Check subscription status
# ========================================
sub_status = dz.get_subscription_request_details(
    domainIdentifier=domain.id,
    identifier=subscription['id']
)
print(f"Subscription status: {sub_status['status']}")

# Wait for approval (in production, use EventBridge notification)
# For demo: assume approved

# ========================================
# 5. Query the data with awswrangler (no 10k row limit!)
# ========================================
print("\nQuerying data from approved subscription...")

df = wr.athena.read_sql_query(
    sql="""
        SELECT 
            customer_id,
            customer_name,
            total_purchases,
            last_purchase_date,
            customer_segment
        FROM sales_db.customers
        WHERE last_purchase_date >= DATE '2024-01-01'
        ORDER BY total_purchases DESC
        LIMIT 100000
    """,
    database="sales_db",
    ctas_approach=False  # Direct query without CTAS
)

print(f"Retrieved {len(df)} rows (no 10k limit!)")
print(df.head())

# ========================================
# 6. Perform data analysis
# ========================================
print("\nPerforming analysis...")

# Segment analysis
segment_stats = df.groupby('customer_segment').agg({
    'total_purchases': ['count', 'mean', 'sum'],
    'customer_id': 'count'
}).round(2)

print("\nSegment Analysis:")
print(segment_stats)

# ========================================
# 7. Store processed results back to S3
# ========================================
output_path = 's3://ml-platform-bucket/data/processed/customer_analysis/'

wr.s3.to_parquet(
    df=segment_stats,
    path=output_path,
    dataset=True,
    mode='overwrite',
    database='processed_db',
    table='customer_segments'
)

print(f"\nResults stored at: {output_path}")
print("Data cataloged in: processed_db.customer_segments")

# ========================================
# 8. Query local DataFrame using DuckDB (via sqlutils)
# ========================================
from sagemaker_studio import sqlutils

# Query the pandas DataFrame directly with SQL
top_customers = sqlutils.sql("""
    SELECT 
        customer_segment,
        COUNT(*) as customer_count,
        AVG(total_purchases) as avg_purchases
    FROM df
    GROUP BY customer_segment
    ORDER BY avg_purchases DESC
""")

print("\nTop customer segments:")
print(top_customers)
```

---

## 📦 Complete Example 2: Model Training Workflow with Catalog Integration

```python
"""
Complete ML workflow: Access data → Prepare features → Train model → Register
"""

import boto3
from sagemaker_studio import Domain, Project, sql
import awswrangler as wr
import sagemaker
from sagemaker.estimator import Estimator
from sagemaker.inputs import TrainingInput
import pandas as pd

# ========================================
# 1. Setup
# ========================================
domain = Domain()
project = Project()
sm_session = sagemaker.Session()
role = sagemaker.get_execution_role()

print(f"Project: {project.name}")
print(f"Execution Role: {role}")

# ========================================
# 2. Load training data from catalog
# ========================================
athena_conn = project.connection('project.athena')

# Query with awswrangler (full dataset, no limits)
train_df = wr.athena.read_sql_query(
    sql="""
        SELECT 
            customer_id,
            feature_1,
            feature_2,
            feature_3,
            target_variable
        FROM processed_db.ml_features
        WHERE dataset_split = 'train'
    """,
    database="processed_db"
)

print(f"Loaded {len(train_df)} training samples")

# ========================================
# 3. Feature engineering
# ========================================
# Create derived features
train_df['feature_interaction'] = train_df['feature_1'] * train_df['feature_2']
train_df['feature_ratio'] = train_df['feature_1'] / (train_df['feature_3'] + 1)

# ========================================
# 4. Store processed features to S3
# ========================================
train_path = 's3://ml-platform-bucket/data/features/train/'

wr.s3.to_csv(
    df=train_df,
    path=train_path + 'train.csv',
    index=False
)

print(f"Training data stored at: {train_path}")

# ========================================
# 5. Create SageMaker training job
# ========================================
from sagemaker.sklearn.estimator import SKLearn

sklearn_estimator = SKLearn(
    entry_point='train.py',
    source_dir='./scripts',
    role=role,
    instance_type='ml.m5.xlarge',
    framework_version='1.2-1',
    py_version='py3',
    hyperparameters={
        'max_depth': 10,
        'n_estimators': 100
    },
    output_path='s3://ml-platform-bucket/artifacts/models/',
    base_job_name='customer-model'
)

# ========================================
# 6. Train model
# ========================================
train_input = TrainingInput(
    s3_data=train_path,
    content_type='text/csv'
)

sklearn_estimator.fit({'train': train_input})

print(f"Training job: {sklearn_estimator.latest_training_job.name}")

# ========================================
# 7. Register model in Model Registry
# ========================================
model_package = sklearn_estimator.register(
    content_types=["text/csv"],
    response_types=["text/csv"],
    inference_instances=["ml.t2.medium", "ml.m5.large"],
    transform_instances=["ml.m5.large"],
    model_package_group_name="customer-models",
    approval_status="PendingManualApproval",
    description="Customer segmentation model v1.0"
)

print(f"Model registered: {model_package.model_package_arn}")

# ========================================
# 8. Track experiment in SageMaker Experiments
# ========================================
from sagemaker.experiments import Run

with Run(
    experiment_name="customer-segmentation",
    run_name=f"training-{sklearn_estimator.latest_training_job.name}",
    sagemaker_session=sm_session
) as run:
    # Log parameters
    run.log_parameters({
        "max_depth": 10,
        "n_estimators": 100,
        "training_samples": len(train_df)
    })
    
    # Log metrics (would come from training)
    run.log_metric(name="accuracy", value=0.92)
    run.log_metric(name="f1_score", value=0.89)
    
    # Log model artifact
    run.log_artifact(
        name="model",
        value=sklearn_estimator.model_data,
        media_type="application/x-tar"
    )

print("Experiment tracked successfully")
```

---

## 📦 Complete Example 3: Subscription Approval Automation

```python
"""
Automate subscription approvals using EventBridge + Lambda
This would run as a Lambda function triggered by DataZone events
"""

import json
import boto3
from typing import Dict, Any

dz = boto3.client('datazone')
sns = boto3.client('sns')

def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Lambda function to auto-approve or route subscription requests
    Triggered by EventBridge when subscription request is created
    """
    
    print(f"Received event: {json.dumps(event)}")
    
    # Extract DataZone event details
    detail = event['detail']
    domain_id = detail['domainId']
    request_id = detail['subscriptionRequestId']
    requester = detail['createdBy']
    
    # Get subscription request details
    request = dz.get_subscription_request_details(
        domainIdentifier=domain_id,
        identifier=request_id
    )
    
    print(f"Subscription request: {request_id}")
    print(f"Requester: {requester}")
    print(f"Reason: {request.get('requestReason', 'No reason provided')}")
    
    # ========================================
    # Auto-approval logic
    # ========================================
    
    # Rule 1: Auto-approve if requester is in approved group
    approved_groups = ['data-scientists', 'ml-engineers']
    requester_groups = get_user_groups(requester)  # Custom function
    
    should_auto_approve = bool(set(requester_groups) & set(approved_groups))
    
    # Rule 2: Check if asset is in public catalog
    asset_listing = request['subscribedListings'][0]
    asset_details = dz.get_listing(
        domainIdentifier=domain_id,
        identifier=asset_listing['identifier']
    )
    
    is_public = asset_details.get('isPublic', False)
    
    # ========================================
    # Decision logic
    # ========================================
    
    if should_auto_approve or is_public:
        # Auto-approve
        print(f"Auto-approving request {request_id}")
        
        response = dz.accept_subscription_request(
            domainIdentifier=domain_id,
            identifier=request_id,
            decisionComment='Auto-approved: User in approved group'
        )
        
        # Notify requester
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789012:datazone-approvals',
            Subject=f'Subscription Approved: {request_id}',
            Message=f"""
Your subscription request has been auto-approved.
Request ID: {request_id}
Asset: {asset_listing['identifier']}
You can now access the data in your project.
            """
        )
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'action': 'auto-approved',
                'request_id': request_id
            })
        }
    
    else:
        # Route to manual approval
        print(f"Routing request {request_id} for manual approval")
        
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789012:datazone-manual-review',
            Subject=f'Manual Review Required: {request_id}',
            Message=f"""
A subscription request requires manual review.

Request ID: {request_id}
Requester: {requester}
Reason: {request.get('requestReason')}
Asset: {asset_listing['identifier']}

Review at: https://console.aws.amazon.com/datazone/
            """
        )
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'action': 'manual-review-required',
                'request_id': request_id
            })
        }


def get_user_groups(user_id: str) -> list:
    """Helper function to get user's groups"""
    # In production, query IAM or DataZone for user groups
    # Simplified for example
    return ['data-scientists']


# ========================================
# EventBridge Rule Configuration (CDK/CloudFormation)
# ========================================
"""
{
  "source": ["aws.datazone"],
  "detail-type": ["Subscription Request Status Change"],
  "detail": {
    "status": ["PENDING"]
  }
}
"""
```

---

## 📦 Complete Example 4: Batch Inference with Result Cataloging

```python
"""
Complete batch inference workflow:
1. Prepare batch input
2. Submit batch transform job
3. Monitor completion
4. Catalog results with Glue Crawler
5. Query results with Athena
"""

import boto3
import time
from datetime import datetime
import pandas as pd
import awswrangler as wr

# ========================================
# Setup
# ========================================
sm = boto3.client('sagemaker')
s3 = boto3.client('s3')
glue = boto3.client('glue')

bucket = 'ml-platform-bucket'
model_name = 'customer-model-prod'
timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')

# ========================================
# 1. Prepare batch input data
# ========================================
print("Preparing batch input data...")

# Query data to score
input_df = wr.athena.read_sql_query(
    sql="""
        SELECT 
            customer_id,
            feature_1,
            feature_2,
            feature_3
        FROM processed_db.customers_to_score
        WHERE score_date = CURRENT_DATE
    """,
    database="processed_db"
)

print(f"Scoring {len(input_df)} customers")

# Save to S3 batch input location
input_path = f's3://{bucket}/inference/batch/input/batch_{timestamp}.csv'
output_path = f's3://{bucket}/inference/batch/output/batch_{timestamp}/'

wr.s3.to_csv(
    df=input_df,
    path=input_path,
    index=False
)

print(f"Input data stored at: {input_path}")

# ========================================
# 2. Create and start batch transform job
# ========================================
print("\nStarting batch transform job...")

job_name = f'customer-scoring-{timestamp}'

transform_params = {
    'TransformJobName': job_name,
    'ModelName': model_name,
    'TransformInput': {
        'DataSource': {
            'S3DataSource': {
                'S3DataType': 'S3Prefix',
                'S3Uri': input_path
            }
        },
        'ContentType': 'text/csv',
        'SplitType': 'Line'
    },
    'TransformOutput': {
        'S3OutputPath': output_path,
        'Accept': 'text/csv',
        'AssembleWith': 'Line'
    },
    'TransformResources': {
        'InstanceType': 'ml.m5.xlarge',
        'InstanceCount': 2
    },
    'BatchStrategy': 'MultiRecord',
    'MaxPayloadInMB': 6,
    'MaxConcurrentTransforms': 4
}

response = sm.create_transform_job(**transform_params)

print(f"Transform job created: {job_name}")

# ========================================
# 3. Monitor job completion
# ========================================
print("\nMonitoring job progress...")

while True:
    job_desc = sm.describe_transform_job(TransformJobName=job_name)
    status = job_desc['TransformJobStatus']
    
    print(f"Status: {status}")
    
    if status in ['Completed', 'Failed', 'Stopped']:
        break
    
    time.sleep(30)

if status == 'Completed':
    print(f"\n✓ Transform job completed successfully")
    print(f"Output location: {output_path}")
else:
    print(f"\n✗ Transform job {status}")
    print(f"Failure reason: {job_desc.get('FailureReason', 'Unknown')}")
    exit(1)

# ========================================
# 4. Catalog results with Glue Crawler
# ========================================
print("\nCataloging results with Glue Crawler...")

crawler_name = 'batch-results-crawler'

# Check if crawler exists, create if not
try:
    glue.get_crawler(Name=crawler_name)
    print(f"Using existing crawler: {crawler_name}")
except glue.exceptions.EntityNotFoundException:
    print(f"Creating new crawler: {crawler_name}")
    
    glue.create_crawler(
        Name=crawler_name,
        Role='arn:aws:iam::123456789012:role/GlueCrawlerRole',
        DatabaseName='inference_results',
        Targets={
            'S3Targets': [{
                'Path': f's3://{bucket}/inference/batch/output/'
            }]
        },
        SchemaChangePolicy={
            'UpdateBehavior': 'UPDATE_IN_DATABASE',
            'DeleteBehavior': 'LOG'
        }
    )

# Start crawler
glue.start_crawler(Name=crawler_name)

print(f"Crawler started: {crawler_name}")

# Wait for crawler to complete
while True:
    crawler_status = glue.get_crawler(Name=crawler_name)
    state = crawler_status['Crawler']['State']
    
    print(f"Crawler state: {state}")
    
    if state == 'READY':
        break
    
    time.sleep(15)

print("✓ Results cataloged successfully")

# ========================================
# 5. Query results with Athena
# ========================================
print("\nQuerying batch results...")

results_df = wr.athena.read_sql_query(
    sql=f"""
        SELECT 
            customer_id,
            prediction,
            confidence_score
        FROM inference_results.batch_{timestamp.replace('-', '_')}
        WHERE confidence_score > 0.8
        ORDER BY confidence_score DESC
        LIMIT 100
    """,
    database="inference_results"
)

print(f"\nTop 10 high-confidence predictions:")
print(results_df.head(10))

# ========================================
# 6. Store summary statistics
# ========================================
summary = {
    'job_name': job_name,
    'timestamp': timestamp,
    'input_records': len(input_df),
    'output_records': len(results_df),
    'high_confidence_predictions': len(results_df[results_df['confidence_score'] > 0.9]),
    'avg_confidence': results_df['confidence_score'].mean()
}

print(f"\nBatch Job Summary:")
for key, value in summary.items():
    print(f"  {key}: {value}")

# Store summary
summary_path = f's3://{bucket}/inference/batch/summaries/summary_{timestamp}.json'
wr.s3.to_json(
    df=pd.DataFrame([summary]),
    path=summary_path
)

print(f"\nSummary stored at: {summary_path}")
```

---

## 📦 Complete Example 5: Real-time Inference with Model Monitoring

```python
"""
Deploy model with real-time endpoint and enable model monitoring
"""

import boto3
import sagemaker
from sagemaker.model import Model
from sagemaker.predictor import Predictor
from sagemaker.model_monitor import DataCaptureConfig, DefaultModelMonitor
from sagemaker_studio import Project
import json

# ========================================
# Setup
# ========================================
project = Project()
sm_session = sagemaker.Session()
role = sagemaker.get_execution_role()
sm = boto3.client('sagemaker')

print(f"Project: {project.name}")

# ========================================
# 1. Get approved model from registry
# ========================================
model_package_arn = 'arn:aws:sagemaker:us-east-1:123456789012:model-package/customer-models/1'

# Create model from registry
model = Model(
    image_uri=None,  # Will use image from model package
    model_data=None,  # Will use data from model package
    role=role,
    sagemaker_session=sm_session,
    model_package_arn=model_package_arn
)

# ========================================
# 2. Configure data capture for monitoring
# ========================================
data_capture_config = DataCaptureConfig(
    enable_capture=True,
    sampling_percentage=100,  # Capture 100% of requests
    destination_s3_uri=f's3://ml-platform-bucket/monitoring/data-capture',
    capture_options=["REQUEST", "RESPONSE"]
)

# ========================================
# 3. Deploy model with monitoring enabled
# ========================================
print("\nDeploying model to real-time endpoint...")

predictor = model.deploy(
    instance_type='ml.m5.large',
    initial_instance_count=1,
    endpoint_name='customer-model-prod',
    data_capture_config=data_capture_config,
    wait=True
)

print(f"✓ Endpoint deployed: {predictor.endpoint_name}")

# ========================================
# 4. Configure auto-scaling
# ========================================
asg = boto3.client('application-autoscaling')

# Register scalable target
asg.register_scalable_target(
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{predictor.endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    MinCapacity=1,
    MaxCapacity=5
)

# Configure scaling policy
asg.put_scaling_policy(
    PolicyName='sagemaker-scaling-policy',
    ServiceNamespace='sagemaker',
    ResourceId=f'endpoint/{predictor.endpoint_name}/variant/AllTraffic',
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    PolicyType='TargetTrackingScaling',
    TargetTrackingScalingPolicyConfiguration={
        'TargetValue': 70.0,  # Target 70% invocations per instance
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'SageMakerVariantInvocationsPerInstance'
        },
        'ScaleInCooldown': 300,
        'ScaleOutCooldown': 60
    }
)

print("✓ Auto-scaling configured (1-5 instances)")

# ========================================
# 5. Set up model monitoring
# ========================================
print("\nConfiguring model monitoring...")

monitor = DefaultModelMonitor(
    role=role,
    instance_count=1,
    instance_type='ml.m5.xlarge',
    volume_size_in_gb=20,
    max_runtime_in_seconds=3600,
    sagemaker_session=sm_session
)

# Create baseline from training data
baseline_results = monitor.suggest_baseline(
    baseline_dataset='s3://ml-platform-bucket/data/features/train/train.csv',
    dataset_format=sagemaker.model_monitor.DatasetFormat.csv(header=True),
    output_s3_uri='s3://ml-platform-bucket/monitoring/baseline',
    wait=True
)

print("✓ Baseline created")

# Create monitoring schedule
monitor.create_monitoring_schedule(
    monitor_schedule_name='customer-model-monitor',
    endpoint_input=predictor.endpoint_name,
    output_s3_uri='s3://ml-platform-bucket/monitoring/results',
    statistics=baseline_results.baseline_statistics,
    constraints=baseline_results.constraint_violations,
    schedule_cron_expression='cron(0 * * * ? *)',  # Hourly
    enable_cloudwatch_metrics=True
)

print("✓ Monitoring schedule created (hourly)")

# ========================================
# 6. Test endpoint with sample prediction
# ========================================
print("\nTesting endpoint...")

test_data = {
    'customer_id': 'CUST_001',
    'feature_1': 25.5,
    'feature_2': 100.0,
    'feature_3': 5.2
}

# Make prediction
result = predictor.predict(
    data=json.dumps(test_data),
    initial_args={'ContentType': 'application/json'}
)

print(f"Prediction result: {result}")

# ========================================
# 7. Create CloudWatch dashboard
# ========================================
cw = boto3.client('cloudwatch')

dashboard_body = {
    "widgets": [
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/SageMaker", "ModelLatency", {"stat": "Average"}],
                    [".", ".", {"stat": "p99"}]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-east-1",
                "title": "Model Latency"
            }
        },
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/SageMaker", "Invocations", {"stat": "Sum"}],
                    [".", "Invocation4XXErrors", {"stat": "Sum"}],
                    [".", "Invocation5XXErrors", {"stat": "Sum"}]
                ],
                "period": 300,
                "stat": "Sum",
                "region": "us-east-1",
                "title": "Invocations"
            }
        }
    ]
}

cw.put_dashboard(
    DashboardName='CustomerModelMonitoring',
    DashboardBody=json.dumps(dashboard_body)
)

print("✓ CloudWatch dashboard created")

print("\n=== Deployment Complete ===")
print(f"Endpoint: {predictor.endpoint_name}")
print(f"Monitoring: Enabled (hourly checks)")
print(f"Auto-scaling: 1-5 instances")
print(f"Data capture: 100% of requests")
print(f"Dashboard: https://console.aws.amazon.com/cloudwatch/home#dashboards:name=CustomerModelMonitoring")
```

---

## 🔧 Advanced Patterns & Best Practices

### Pattern 1: Connection Management

```python
from sagemaker_studio import Project

project = Project()

# Get all available connections
connections = project.list_connections()
print(f"Available connections: {connections}")

# Get specific connection
athena_conn = project.connection('project.athena')
redshift_conn = project.connection('project.redshift')

# Create service clients using connections
athena_client = athena_conn.create_client('athena')
redshift_data_client = redshift_conn.create_client('redshift-data')

# Use environment ID for other operations
env_id = athena_conn.environment_id
print(f"Environment ID: {env_id}")
```

### Pattern 2: Efficient Data Loading

```python
import awswrangler as wr

# Bad: Multiple small queries (slow)
dfs = []
for month in range(1, 13):
    df = wr.athena.read_sql_query(
        sql=f"SELECT * FROM table WHERE month = {month}",
        database="db"
    )
    dfs.append(df)
result = pd.concat(dfs)

# Good: Single query with filter (fast)
result = wr.athena.read_sql_query(
    sql="SELECT * FROM table WHERE month BETWEEN 1 AND 12",
    database="db"
)

# Best: Partitioned read (fastest)
result = wr.athena.read_sql_table(
    table="table",
    database="db",
    partition_filter=lambda x: x["month"] >= "01" and x["month"] <= "12"
)
```

### Pattern 3: Incremental Data Processing

```python
from datetime import datetime, timedelta
import awswrangler as wr

# Get last processed date from metadata
last_processed = get_last_processed_date()  # Custom function

# Process only new data
new_data = wr.athena.read_sql_query(
    sql=f"""
        SELECT *
        FROM source_table
        WHERE process_date > DATE '{last_processed}'
    """,
    database="raw_db"
)

# Process and store
processed = process_data(new_data)

wr.s3.to_parquet(
    df=processed,
    path='s3://bucket/processed/',
    dataset=True,
    mode='append',  # Append new data
    partition_cols=['year', 'month', 'day']
)

# Update metadata
update_last_processed_date(datetime.now())
```

### Pattern 4: Error Handling

```python
from botocore.exceptions import ClientError
import time

def safe_datazone_operation(func, max_retries=3):
    """Retry wrapper for DataZone operations"""
    for attempt in range(max_retries):
        try:
            return func()
        except ClientError as e:
            error_code = e.response['Error']['Code']
            
            if error_code == 'ThrottlingException':
                # Exponential backoff
                wait_time = 2 ** attempt
                print(f"Throttled, retrying in {wait_time}s...")
                time.sleep(wait_time)
            elif error_code == 'ResourceNotFoundException':
                print(f"Resource not found: {e}")
                return None
            else:
                raise
    
    raise Exception(f"Max retries ({max_retries}) exceeded")

# Usage
result = safe_datazone_operation(
    lambda: dz.get_subscription(
        domainIdentifier='dzd_xxx',
        identifier='sub_id'
    )
)
```

---

## 📚 Additional Resources & Documentation

### Official Documentation
- **SMUS User Guide**: https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/
- **DataZone API Reference**: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/datazone.html
- **AWS SDK for pandas**: https://aws-sdk-pandas.readthedocs.io/
- **SageMaker Python SDK**: https://sagemaker.readthedocs.io/

### Blog Posts & Tutorials
- **Custom Subscription Workflows**: https://aws.amazon.com/blogs/big-data/implement-a-custom-subscription-workflow-for-unmanaged-amazon-s3-assets-publishe