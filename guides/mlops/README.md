# MLOps (Machine Learning Operations) for DevOps

## 🎯 Overview

MLOps combines Machine Learning, DevOps, and Data Engineering to deploy and maintain ML systems in production reliably and efficiently. For DevOps engineers, understanding MLOps is essential for supporting data science teams and ML infrastructure.

**Duration:** 2 weeks (Bonus Content)
**Time Investment:** 4-5 hours/day
**Difficulty:** Advanced
**Prerequisites:** Python, Docker, Kubernetes, CI/CD basics

---

## 📚 What You'll Learn

### Week 1: MLOps Fundamentals
- ML lifecycle overview
- Model training and versioning
- Experiment tracking
- Model registry
- Feature stores

### Week 2: Production ML Systems
- Model serving and deployment
- Monitoring and observability
- CI/CD for ML
- Model retraining pipelines
- ML infrastructure

---

## 🎓 Important Concepts

### 1. MLOps Lifecycle

```
MLOps Lifecycle
├─ Data Engineering
│  ├─ Data Collection
│  ├─ Data Validation
│  ├─ Data Preprocessing
│  └─ Feature Engineering
│
├─ Model Development
│  ├─ Experiment Tracking
│  ├─ Model Training
│  ├─ Hyperparameter Tuning
│  └─ Model Evaluation
│
├─ Model Deployment
│  ├─ Model Packaging
│  ├─ Model Serving
│  ├─ A/B Testing
│  └─ Canary Deployment
│
├─ Monitoring & Maintenance
│  ├─ Model Performance
│  ├─ Data Drift Detection
│  ├─ Model Retraining
│  └─ Incident Response
│
└─ Governance
   ├─ Model Versioning
   ├─ Reproducibility
   ├─ Compliance
   └─ Audit Trails
```

**Key Differences from Traditional DevOps:**
- Data versioning alongside code
- Model performance degradation over time
- Experiment tracking and reproducibility
- Feature engineering pipelines
- A/B testing for models

---

### 2. ML Infrastructure Stack

```
MLOps Technology Stack
├─ Data Layer
│  ├─ Data Lakes (S3, Azure Data Lake, GCS)
│  ├─ Data Warehouses (Snowflake, BigQuery)
│  ├─ Feature Stores (Feast, Tecton)
│  └─ Data Versioning (DVC, LakeFS)
│
├─ Experiment Tracking
│  ├─ MLflow
│  ├─ Weights & Biases
│  ├─ Neptune.ai
│  └─ TensorBoard
│
├─ Model Training
│  ├─ Kubeflow
│  ├─ SageMaker
│  ├─ Azure ML
│  └─ Vertex AI
│
├─ Model Registry
│  ├─ MLflow Model Registry
│  ├─ SageMaker Model Registry
│  └─ Custom Solutions
│
├─ Model Serving
│  ├─ TensorFlow Serving
│  ├─ TorchServe
│  ├─ Seldon Core
│  ├─ KServe
│  └─ BentoML
│
├─ Monitoring
│  ├─ Prometheus + Grafana
│  ├─ Evidently AI
│  ├─ Arize AI
│  └─ WhyLabs
│
└─ Orchestration
   ├─ Airflow
   ├─ Kubeflow Pipelines
   ├─ Prefect
   └─ Metaflow
```

---

### 3. Experiment Tracking

```
Experiment Tracking Components
├─ Parameters
│  ├─ Hyperparameters
│  ├─ Model architecture
│  └─ Training configuration
│
├─ Metrics
│  ├─ Training metrics
│  ├─ Validation metrics
│  └─ Test metrics
│
├─ Artifacts
│  ├─ Model files
│  ├─ Plots and visualizations
│  └─ Datasets
│
└─ Metadata
   ├─ Git commit hash
   ├─ Environment info
   └─ Timestamps
```

**MLflow Example:**
```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, f1_score

# Set tracking URI
mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("customer-churn-prediction")

# Start MLflow run
with mlflow.start_run(run_name="rf-experiment-1"):
    # Log parameters
    n_estimators = 100
    max_depth = 10
    mlflow.log_param("n_estimators", n_estimators)
    mlflow.log_param("max_depth", max_depth)
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=n_estimators,
        max_depth=max_depth,
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Make predictions
    y_pred = model.predict(X_test)
    
    # Log metrics
    accuracy = accuracy_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred, average='weighted')
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("f1_score", f1)
    
    # Log model
    mlflow.sklearn.log_model(
        model,
        "model",
        registered_model_name="customer-churn-rf"
    )
    
    # Log artifacts
    import matplotlib.pyplot as plt
    from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
    
    cm = confusion_matrix(y_test, y_pred)
    disp = ConfusionMatrixDisplay(cm)
    disp.plot()
    plt.savefig("confusion_matrix.png")
    mlflow.log_artifact("confusion_matrix.png")
    
    # Log dataset
    mlflow.log_artifact("data/train.csv")
    
    print(f"Run ID: {mlflow.active_run().info.run_id}")
```

---

### 4. Model Serving Patterns

```
Model Serving Architectures
├─ Batch Prediction
│  └─ Scheduled jobs processing large datasets
│
├─ Real-time Prediction
│  ├─ REST API
│  ├─ gRPC
│  └─ WebSocket
│
├─ Streaming Prediction
│  └─ Kafka/Kinesis integration
│
└─ Edge Deployment
   └─ Mobile/IoT devices
```

**Model Serving with FastAPI:**
```python
from fastapi import FastAPI
from pydantic import BaseModel
import mlflow.pyfunc
import numpy as np

app = FastAPI()

# Load model
model = mlflow.pyfunc.load_model("models:/customer-churn-rf/production")

class PredictionRequest(BaseModel):
    features: list[float]

class PredictionResponse(BaseModel):
    prediction: int
    probability: float

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    # Prepare input
    input_data = np.array([request.features])
    
    # Make prediction
    prediction = model.predict(input_data)[0]
    probability = model.predict_proba(input_data)[0][prediction]
    
    return PredictionResponse(
        prediction=int(prediction),
        probability=float(probability)
    )

@app.get("/health")
async def health():
    return {"status": "healthy"}

@app.get("/model/info")
async def model_info():
    return {
        "model_name": "customer-churn-rf",
        "version": "production",
        "framework": "sklearn"
    }
```

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-model-serving
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-model
  template:
    metadata:
      labels:
        app: ml-model
        version: v1
    spec:
      containers:
      - name: model-server
        image: ml-model:v1
        ports:
        - containerPort: 8000
        env:
        - name: MLFLOW_TRACKING_URI
          value: "http://mlflow:5000"
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: ml-model-service
spec:
  selector:
    app: ml-model
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

---

### 5. Feature Store

```
Feature Store Architecture
├─ Feature Definition
│  ├─ Feature schema
│  ├─ Transformation logic
│  └─ Data sources
│
├─ Feature Computation
│  ├─ Batch features
│  ├─ Streaming features
│  └─ On-demand features
│
├─ Feature Storage
│  ├─ Online store (low latency)
│  └─ Offline store (training)
│
└─ Feature Serving
   ├─ Training data generation
   └─ Real-time feature retrieval
```

**Feast Feature Store Example:**
```python
# feature_repo/features.py
from feast import Entity, Feature, FeatureView, FileSource, ValueType
from datetime import timedelta

# Define entity
customer = Entity(
    name="customer_id",
    value_type=ValueType.INT64,
    description="Customer ID"
)

# Define data source
customer_source = FileSource(
    path="data/customer_features.parquet",
    event_timestamp_column="event_timestamp"
)

# Define feature view
customer_features = FeatureView(
    name="customer_features",
    entities=["customer_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="age", dtype=ValueType.INT64),
        Feature(name="total_purchases", dtype=ValueType.INT64),
        Feature(name="avg_purchase_value", dtype=ValueType.DOUBLE),
        Feature(name="days_since_last_purchase", dtype=ValueType.INT64),
    ],
    online=True,
    source=customer_source,
    tags={"team": "ml-platform"}
)

# Usage in training
from feast import FeatureStore
import pandas as pd

store = FeatureStore(repo_path=".")

# Get training data
entity_df = pd.DataFrame({
    "customer_id": [1001, 1002, 1003],
    "event_timestamp": [
        pd.Timestamp("2024-01-01"),
        pd.Timestamp("2024-01-01"),
        pd.Timestamp("2024-01-01")
    ]
})

training_df = store.get_historical_features(
    entity_df=entity_df,
    features=[
        "customer_features:age",
        "customer_features:total_purchases",
        "customer_features:avg_purchase_value",
        "customer_features:days_since_last_purchase"
    ]
).to_df()

# Usage in serving
features = store.get_online_features(
    features=[
        "customer_features:age",
        "customer_features:total_purchases"
    ],
    entity_rows=[{"customer_id": 1001}]
).to_dict()
```

---

### 6. Model Monitoring

```
Model Monitoring Metrics
├─ Performance Metrics
│  ├─ Accuracy, Precision, Recall
│  ├─ F1 Score, AUC-ROC
│  └─ Business metrics (revenue, conversions)
│
├─ Data Quality
│  ├─ Missing values
│  ├─ Outliers
│  └─ Schema violations
│
├─ Data Drift
│  ├─ Feature distribution changes
│  ├─ Statistical tests (KS, Chi-square)
│  └─ Population stability index
│
├─ Concept Drift
│  ├─ Target distribution changes
│  └─ Model performance degradation
│
└─ System Metrics
   ├─ Latency
   ├─ Throughput
   └─ Error rates
```

**Monitoring with Evidently:**
```python
from evidently.dashboard import Dashboard
from evidently.tabs import DataDriftTab, CatTargetDriftTab
import pandas as pd

# Load reference and current data
reference_data = pd.read_csv("reference_data.csv")
current_data = pd.read_csv("current_data.csv")

# Create dashboard
dashboard = Dashboard(tabs=[
    DataDriftTab(),
    CatTargetDriftTab()
])

# Generate report
dashboard.calculate(
    reference_data=reference_data,
    current_data=current_data,
    column_mapping={
        'target': 'churn',
        'prediction': 'prediction',
        'numerical_features': ['age', 'total_purchases'],
        'categorical_features': ['gender', 'subscription_type']
    }
)

# Save report
dashboard.save("drift_report.html")

# Programmatic drift detection
from evidently.model_profile import Profile
from evidently.model_profile.sections import DataDriftProfileSection

profile = Profile(sections=[DataDriftProfileSection()])
profile.calculate(reference_data, current_data)

drift_report = profile.json()
if drift_report['data_drift']['data_drift_detected']:
    print("Data drift detected! Triggering model retraining...")
    # Trigger retraining pipeline
```

---

### 7. CI/CD for ML

```
ML CI/CD Pipeline
├─ Code Changes
│  └─ Git commit
│
├─ CI Pipeline
│  ├─ Code quality checks
│  ├─ Unit tests
│  ├─ Data validation
│  ├─ Model training
│  ├─ Model evaluation
│  └─ Model testing
│
├─ CD Pipeline
│  ├─ Model packaging
│  ├─ Deploy to staging
│  ├─ Integration tests
│  ├─ A/B testing setup
│  └─ Deploy to production
│
└─ Continuous Training
   ├─ Monitor performance
   ├─ Detect drift
   ├─ Trigger retraining
   └─ Auto-deploy if improved
```

**GitHub Actions for ML:**
```yaml
name: ML CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: |
          pytest tests/ --cov=src --cov-report=xml
      
      - name: Validate data
        run: |
          python scripts/validate_data.py
  
  train:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Train model
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
        run: |
          python src/train.py \
            --data-path data/train.csv \
            --experiment-name customer-churn \
            --run-name "automated-training-${{ github.sha }}"
      
      - name: Evaluate model
        run: |
          python src/evaluate.py \
            --model-uri "models:/customer-churn-rf/latest" \
            --test-data data/test.csv \
            --threshold 0.85
  
  deploy:
    needs: train
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker image
        run: |
          docker build -t ml-model:${{ github.sha }} .
          docker tag ml-model:${{ github.sha }} ml-model:latest
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push ml-model:${{ github.sha }}
          docker push ml-model:latest
      
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v1
        with:
          manifests: |
            k8s/deployment.yaml
            k8s/service.yaml
          images: ml-model:${{ github.sha }}
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Set Up MLflow Tracking Server (45 minutes)

**Objective:** Deploy MLflow for experiment tracking

```bash
# 1. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: mlflow
      POSTGRES_PASSWORD: mlflow
      POSTGRES_DB: mlflow
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    volumes:
      - minio-data:/data
    ports:
      - "9000:9000"
      - "9001:9001"
  
  mlflow:
    image: python:3.9
    command: >
      bash -c "pip install mlflow psycopg2-binary boto3 &&
               mlflow server
               --backend-store-uri postgresql://mlflow:mlflow@postgres:5432/mlflow
               --default-artifact-root s3://mlflow/
               --host 0.0.0.0
               --port 5000"
    environment:
      AWS_ACCESS_KEY_ID: minioadmin
      AWS_SECRET_ACCESS_KEY: minioadmin
      MLFLOW_S3_ENDPOINT_URL: http://minio:9000
    ports:
      - "5000:5000"
    depends_on:
      - postgres
      - minio

volumes:
  postgres-data:
  minio-data:
EOF

# 2. Start services
docker-compose up -d

# 3. Create MinIO bucket
pip install minio
python << 'PYTHON'
from minio import Minio

client = Minio(
    "localhost:9000",
    access_key="minioadmin",
    secret_key="minioadmin",
    secure=False
)

if not client.bucket_exists("mlflow"):
    client.make_bucket("mlflow")
    print("Bucket 'mlflow' created")
PYTHON

# 4. Test MLflow
python << 'PYTHON'
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("test-experiment")

with mlflow.start_run():
    mlflow.log_param("test_param", "value")
    mlflow.log_metric("test_metric", 0.95)
    print("Test run logged successfully!")
PYTHON

# 5. Access MLflow UI
echo "MLflow UI: http://localhost:5000"
```

**Challenge:**
Set up production MLflow:
1. Add authentication
2. Configure HTTPS
3. Set up backup
4. Add monitoring

---

### Exercise 2: Build ML Training Pipeline (60 minutes)

**Objective:** Create end-to-end training pipeline

```python
# train_pipeline.py
import pandas as pd
import mlflow
import mlflow.sklearn
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
import joblib

def load_data(data_path):
    """Load and validate data"""
    df = pd.read_csv(data_path)
    
    # Data validation
    assert df.shape[0] > 0, "Dataset is empty"
    assert not df.isnull().all().any(), "Column with all null values found"
    
    return df

def preprocess_data(df, target_column):
    """Preprocess data"""
    # Separate features and target
    X = df.drop(columns=[target_column])
    y = df[target_column]
    
    # Handle categorical variables
    X = pd.get_dummies(X, drop_first=True)
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )
    
    # Scale features
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    return X_train_scaled, X_test_scaled, y_train, y_test, scaler, X.columns.tolist()

def train_model(X_train, y_train, params):
    """Train model"""
    model = RandomForestClassifier(**params)
    model.fit(X_train, y_train)
    return model

def evaluate_model(model, X_test, y_test):
    """Evaluate model"""
    y_pred = model.predict(X_test)
    
    metrics = {
        'accuracy': accuracy_score(y_test, y_pred),
        'precision': precision_score(y_test, y_pred, average='weighted'),
        'recall': recall_score(y_test, y_pred, average='weighted'),
        'f1_score': f1_score(y_test, y_pred, average='weighted')
    }
    
    return metrics, y_pred

def main():
    # Configuration
    DATA_PATH = "data/customer_churn.csv"
    TARGET_COLUMN = "churn"
    EXPERIMENT_NAME = "customer-churn-prediction"
    
    # Model parameters
    params = {
        'n_estimators': 100,
        'max_depth': 10,
        'min_samples_split': 5,
        'random_state': 42
    }
    
    # Set MLflow tracking
    mlflow.set_tracking_uri("http://localhost:5000")
    mlflow.set_experiment(EXPERIMENT_NAME)
    
    with mlflow.start_run(run_name="rf-training"):
        # Log parameters
        mlflow.log_params(params)
        
        # Load data
        print("Loading data...")
        df = load_data(DATA_PATH)
        mlflow.log_param("dataset_size", len(df))
        
        # Preprocess
        print("Preprocessing data...")
        X_train, X_test, y_train, y_test, scaler, feature_names = preprocess_data(
            df, TARGET_COLUMN
        )
        mlflow.log_param("n_features", len(feature_names))
        
        # Train
        print("Training model...")
        model = train_model(X_train, y_train, params)
        
        # Evaluate
        print("Evaluating model...")
        metrics, y_pred = evaluate_model(model, X_test, y_test)
        
        # Log metrics
        for metric_name, metric_value in metrics.items():
            mlflow.log_metric(metric_name, metric_value)
            print(f"{metric_name}: {metric_value:.4f}")
        
        # Log model
        mlflow.sklearn.log_model(
            model,
            "model",
            registered_model_name="customer-churn-rf"
        )
        
        # Log scaler
        joblib.dump(scaler, "scaler.pkl")
        mlflow.log_artifact("scaler.pkl")
        
        # Log feature names
        with open("feature_names.txt", "w") as f:
            f.write("\n".join(feature_names))
        mlflow.log_artifact("feature_names.txt")
        
        print(f"\nRun ID: {mlflow.active_run().info.run_id}")
        print("Training completed successfully!")

if __name__ == "__main__":
    main()
```

**Challenge:**
Enhance pipeline with:
1. Hyperparameter tuning
2. Cross-validation
3. Feature importance analysis
4. Model comparison

---

### Exercise 3: Deploy Model with FastAPI (60 minutes)

**Objective:** Create production-ready model serving API

```python
# app.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, validator
import mlflow.pyfunc
import numpy as np
import joblib
from typing import List
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="ML Model API", version="1.0.0")

# Global variables
model = None
scaler = None
feature_names = None

class PredictionRequest(BaseModel):
    features: dict
    
    @validator('features')
    def validate_features(cls, v):
        if not v:
            raise ValueError("Features cannot be empty")
        return v

class PredictionResponse(BaseModel):
    prediction: int
    probability: float
    model_version: str

@app.on_event("startup")
async def load_model():
    """Load model on startup"""
    global model, scaler, feature_names
    
    try:
        # Load model
        model_uri = "models:/customer-churn-rf/production"
        model = mlflow.pyfunc.load_model(model_uri)
        logger.info(f"Model loaded from {model_uri}")
        
        # Load scaler
        scaler = joblib.load("artifacts/scaler.pkl")
        logger.info("Scaler loaded")
        
        # Load feature names
        with open("artifacts/feature_names.txt", "r") as f:
            feature_names = [line.strip() for line in f]
        logger.info(f"Loaded {len(feature_names)} feature names")
        
    except Exception as e:
        logger.error(f"Error loading model: {e}")
        raise

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    """Make prediction"""
    try:
        # Prepare features
        feature_vector = [request.features.get(name, 0) for name in feature_names]
        feature_array = np.array([feature_vector])
        
        # Scale features
        feature_scaled = scaler.transform(feature_array)
        
        # Make prediction
        prediction = int(model.predict(feature_scaled)[0])
        probability = float(model.predict_proba(feature_scaled)[0][prediction])
        
        return PredictionResponse(
            prediction=prediction,
            probability=probability,
            model_version="production"
        )
        
    except Exception as e:
        logger.error(f"Prediction error: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/predict/batch")
async def predict_batch(requests: List[PredictionRequest]):
    """Batch prediction"""
    results = []
    for req in requests:
        result = await predict(req)
        results.append(result)
    return results

@app.get("/health")
async def health():
    """Health check"""
    return {
        "status": "healthy",
        "model_loaded": model is not None
    }

@app.get("/model/info")
async def model_info():
    """Model information"""
    return {
        "model_name": "customer-churn-rf",
        "version": "production",
        "features": feature_names,
        "n_features": len(feature_names)
    }

# Dockerfile
"""
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
COPY artifacts/ artifacts/

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
"""

# requirements.txt
"""
fastapi==0.104.1
uvicorn==0.24.0
mlflow==2.8.0
scikit-learn==1.3.2
numpy==1.24.3
pydantic==2.5.0
joblib==1.3.2
"""
```

**Challenge:**
Add advanced features:
1. Request validation
2. Rate limiting
3. Caching
4. Async processing

---

## 🎯 Practice Projects

### Project 1: End-to-End ML Platform
Build complete ML platform:
- Data pipeline with Airflow
- Feature store with Feast
- Experiment tracking with MLflow
- Model serving with KServe
- Monitoring with Prometheus/Grafana
- CI/CD with GitHub Actions

### Project 2: Real-time Recommendation System
Deploy recommendation engine:
- Streaming feature computation
- Online model serving
- A/B testing framework
- Performance monitoring
- Auto-scaling infrastructure

### Project 3: Computer Vision Pipeline
Build CV deployment system:
- Image preprocessing pipeline
- Model optimization (TensorRT/ONNX)
- Batch and real-time inference
- Model versioning
- Performance benchmarking

### Project 4: NLP Model Deployment
Deploy language model:
- Text preprocessing pipeline
- Model quantization
- GPU-accelerated serving
- Load balancing
- Cost optimization

---

## 📝 Best Practices

### 1. Reproducibility
- Version control code and data
- Track all experiments
- Document dependencies
- Use containerization
- Seed random numbers
- Save environment configs

### 2. Model Governance
- Implement model registry
- Track model lineage
- Document model cards
- Ensure compliance
- Regular audits
- Version all artifacts

### 3. Monitoring
- Track model performance
- Detect data drift
- Monitor system metrics
- Set up alerts
- Log predictions
- Regular retraining

### 4. Scalability
- Use distributed training
- Implement caching
- Optimize inference
- Auto-scale infrastructure
- Batch predictions
- Use GPU efficiently

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand MLOps lifecycle
- [ ] Set up experiment tracking
- [ ] Implement feature store
- [ ] Create training pipeline
- [ ] Version models

### Week 2
- [ ] Deploy model serving
- [ ] Implement monitoring
- [ ] Set up CI/CD
- [ ] Configure auto-retraining
- [ ] Complete end-to-end project

---

**Next:** Continue with specialized topics or return to [Main README](../../README.md)