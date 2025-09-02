## Overview
This document explains the architecture and deployment workflow of the **Flight Delay Prediction System**. The system integrates **AWS S3**, **Amazon ECS**, **Docker Hub**, **GitHub**, and external data sources such as the **Aviation Stack API**. It supports continuous integration (CI) and continuous delivery (CD), ensuring reliable retraining and deployment of ML models.

---

## Architecture Components

### 1. **S3 Buckets**
- **S3: MODELS**
  - Central storage for all models.
  - Contains:
    - **S3: PROD** → Latest live model version currently serving predictions.
    - **S3: STAGING** → Latest trained model pending evaluation.
    - **Latest.txt** → Keeps track of the key for the latest live model, useful for rollbacks.

- **S3: DATA**
  - Stores datasets used for training and evaluation.
  - Sub-buckets:
    - **S3: TRAINING DATA** → Historical training datasets used for model training.
    - **S3: NEW DATA** → Fresh data fetched daily from the Aviation Stack API. This is used for model evaluation and accuracy measurement.

---

### 2. **Model Training**
- **Google Colab / SageMaker (Training Environment)**
  - Fetches training data from **S3: TRAINING DATA** using **boto3**.
  - Performs:
    - Data preprocessing and cleaning.
    - Model training (Random Forest, Logistic Regression, etc.).
    - Model testing and evaluation.
  - After training:
    - Model is pushed to **S3: STAGING**.
    - Evaluation metrics (accuracy, F1-score) are logged.

---

### 3. **Backend Deployment**
- **Amazon ECS (Elastic Container Service)**
  - Hosts backend services (APIs) inside Docker containers.
  - ECS loads the **latest backend Docker image** from Docker Hub and serves it via a public URL.
  - Backend API integrates with S3 to fetch the **latest model version** and serve real-time predictions.

- **Docker Hub**
  - Stores multiple versions of backend Docker images.
  - Every backend update (after code push) is dockerized and uploaded here.

---

### 4. **Version Control and CI/CD**
- **GitHub**
  - Developers push code into their branches.
  - CI/CD pipeline triggers:
    - Backend and frontend code validation.
    - Dockerization of updated code.
    - Upload of new Docker image to **Docker Hub**.
    - ECS automatically fetches updated image for deployment.

- **Backend Code Responsibilities**
  - **Model Loader** → Loads latest model from S3 (using key in `latest.txt`).
  - **Daily Data Fetcher** → Fetches new data daily via Aviation Stack API and pushes it to **S3: NEW DATA**.
  - **Evaluator** → Runs model tests on **S3: NEW DATA**.  
    - If accuracy > threshold → promotes model from **STAGING** to **PROD** and updates `latest.txt`.
    - If accuracy < threshold → rejects promotion, PROD remains unchanged.

---

### 5. **External API**
- **Aviation Stack API**
  - Provides fresh daily flight data.
  - Integrated in backend service to:
    - Fetch daily data.
    - Update **S3: NEW DATA** for accuracy measurement.

---

## Workflow Summary
1. Training data stored in **S3: TRAINING DATA** is used to train models in Colab/SageMaker.  
2. Trained models are uploaded to **S3: STAGING**.  
3. Backend fetches daily flight data via **Aviation Stack API** and saves it to **S3: NEW DATA**.  
4. Model in **STAGING** is tested on **NEW DATA**.  
   - If accuracy ≥ threshold → promoted to **PROD**, `latest.txt` updated.  
   - If not → rejected.  
5. Backend hosted on **ECS** always loads the PROD model key from **latest.txt** and serves predictions.  
6. Developer code updates go to **GitHub**, triggering Docker builds → Docker Hub → ECS deployment.  

---

## Key Benefits
- **Rollback Capability** → `latest.txt` ensures quick switch to a stable model.  
- **Scalable Design** → Data on S3, compute via ECS, training via SageMaker/Colab.  
- **Automated CI/CD** → Code and models continuously updated without manual intervention.  
- **Clear Separation** → Training, staging, and production environments isolated for safety.  
"""
