---
sidebar_position: 4
sidebar_label: '⚙️ Deployment Guide'
---

# ⚙️ Deployment Guide

This is an advanced MLOps project that requires a configured AWS environment to replicate. This guide provides the high-level steps to deploy the entire pipeline from scratch.

:::info Prerequisites
Before you begin, ensure you have the following:

*   An AWS Account with appropriate IAM permissions to create the resources mentioned below.
*   [AWS CLI](https://aws.amazon.com/cli/) configured locally.
*   [Docker](https://www.docker.com/products/docker-desktop/) installed locally.
*   A [GitHub](https://github.com/) account.
*   The project code cloned to your local machine:
    ```bash
    git clone https://github.com/MdEhsanulHaqueKanan/aws-serverless-youtube-predictor.git
    cd aws-serverless-youtube-predictor
    ```
:::

---

## Deployment Steps

### 1. AWS Infrastructure Setup (Manual)

These resources must be created manually in the AWS Console first.

*   **Create S3 Bucket:** Create a private S3 bucket. This bucket will be used to store the `dataset.csv` file and the trained model artifact (`youtube_popularity_model.joblib`).
*   **Create ECR Repositories:** Create two **private** Amazon ECR repositories:
    *   One for the training image (e.g., `youtube-popularity-trainer`).
    *   One for the inference/predictor image (e.g., `youtube-popularity-predictor`).

### 2. Model Training (One-off Task)

The model is trained as a one-off, containerized task on AWS ECS Fargate.

1.  **Build the Training Docker Image:**
    ```bash
    docker build -t youtube-popularity-trainer -f Dockerfile.train .
    ```
2.  **Push the Image to ECR:** Authenticate Docker to your ECR registry and push the image.
3.  **Run the ECS Task:** Run the training task on a Fargate cluster. This task will read the dataset from S3, train the model, and save the resulting `.joblib` model artifact back to your S3 bucket.

### 3. Set Up the CI/CD Pipeline

The CI/CD pipeline automates the deployment of the prediction API.

1.  **Create CodeBuild Project:** In the AWS Console, create an AWS CodeBuild project.
2.  **Connect to GitHub:** Connect the project to your forked GitHub repository.
3.  **Configure `buildspec.yml`:** The `buildspec.yml` file in the repository contains all the necessary commands. CodeBuild will automatically use it to build the Lambda-compatible container image and push it to your ECR repository.
4.  **Trigger a Build:** A push to the `main` branch on GitHub will automatically trigger a new build in CodeBuild.

### 4. Deploy the Serverless API

The final step is to configure Lambda and API Gateway.

1.  **Create Lambda Function:** Create an AWS Lambda function, selecting the "Container image" option. Use the image that your CodeBuild pipeline pushed to ECR.
2.  **Configure Lambda IAM Role:** Ensure the Lambda function's execution role has IAM permissions to read the model artifact from your S3 bucket.
3.  **Create API Gateway:** Create an Amazon API Gateway (REST API).
4.  **Create Resources and Methods:**
    *   Create a `/predict` resource with a `POST` method.
    *   Integrate the `POST` method with your Lambda function.
    *   Create an `OPTIONS` method and enable CORS to allow the frontend to call the API.
5.  **Deploy the API:** Deploy your API to a stage (e.g., `v1`) to make it public.