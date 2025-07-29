---
sidebar_position: 2
sidebar_label: '🏛️ Architecture'
---

# 🏛️ Project Architecture

This project is composed of several key AWS services working together in two main phases: a CI/CD and Model Training pipeline, and a Serverless Inference pipeline. The diagram below illustrates the complete end-to-end workflow.

```mermaid
graph TD
    subgraph Development & CI/CD
        GitHub[GitHub Repository] --> CodeBuild(AWS CodeBuild)
        CodeBuild --> ECR_Trainer(Amazon ECR<br>Trainer Image)
        CodeBuild --> ECR_Predictor(Amazon ECR<br>Predictor Image)
    end

    subgraph Data & Storage
        S3_Dataset[AWS S3<br>Dataset Bucket]
        S3_Model[AWS S3<br>Model Artifact Bucket]
        S3_Frontend[AWS S3<br>Frontend Hosting]
    end

    subgraph Model Training
        ECS_Fargate(AWS ECS Fargate<br>Model Training)
        ECR_Trainer --> ECS_Fargate
        S3_Dataset -- Reads Data --> ECS_Fargate
        ECS_Fargate -- Saves Model --> S3_Model
    end

    subgraph Inference & API
        Lambda(AWS Lambda<br>Prediction Function)
        ECR_Predictor --> Lambda
        S3_Model -- Loads Model --> Lambda
        APIGateway(Amazon API Gateway) --> Lambda
    end

    subgraph Frontend
        EndUser[End User] --> S3_Frontend
        S3_Frontend -- Calls API --> APIGateway
    end

    subgraph Security & Monitoring
        IAM[AWS IAM]
        CloudWatch[AWS CloudWatch]
        Lambda -- Permissions --> IAM
        ECS_Fargate -- Permissions --> IAM
        CodeBuild -- Permissions --> IAM
        Lambda -- Logs/Metrics --> CloudWatch
        ECS_Fargate -- Logs/Metrics --> CloudWatch
        APIGateway -- Logs/Metrics --> CloudWatch
    end
```

## Component Breakdown

*   **Code:** Source code (`lambda_function.py`, `train.py`) and configuration (`Dockerfile`, `buildspec.yml`) is stored in **GitHub**.
*   **Storage:** The dataset and trained model artifact (`.joblib` file) are stored in **AWS S3**.
*   **Model Training:** A containerized script is run as a one-off task on **AWS ECS Fargate** to train the model and save the artifact to S3.
*   **CI/CD Pipeline:** A push to GitHub triggers **AWS CodeBuild**, which builds a container image for the Lambda function and pushes it to **Amazon ECR**.
*   **Serverless Inference:** **AWS Lambda** runs the container from ECR, serving predictions.
*   **API Layer:** **Amazon API Gateway** provides a public HTTP endpoint that triggers the Lambda function.
*   **Frontend:** A static website (HTML/CSS/JS) is hosted in an **AWS S3** bucket.