# DevOps Notes - Project Nexa

## Opening Introduction

Hi, I am Hamza. I want to walk you through Project Nexa from a DevOps perspective. It is a full-stack exoplanet classification system with a React frontend deployed on Azure Static Web Apps and a FastAPI machine learning backend deployed on Azure Container Apps.

My DevOps focus in this project was containerizing the backend, creating a repeatable Azure deployment flow, enabling automatic frontend deployment through GitHub Actions, and configuring the backend so it can scale based on traffic while staying cost-efficient when idle.

## Key DevOps Talking Points

- The backend runs as a Dockerized FastAPI API on port `8000`.
- The Docker image uses `python:3.11-slim` to keep the base image smaller.
- `requirements.txt` is copied before the app code, so Docker can cache dependency installation.
- Python dependencies are installed with `--no-cache-dir` to reduce image bloat.
- The image includes a healthcheck that calls `/health` every `30s`, with `10s` timeout, `40s` startup delay, and `3` retries.
- The backend loads the CatBoost model, TabNet model, scaler, and encoders once during FastAPI startup.
- The Azure deployment script creates the resource group, Azure Container Registry, Container Apps environment, and the Container App.
- The backend image is built and pushed using `az acr build` as `exoplanet-api:v1`.
- The Container App uses external ingress on port `8000`.
- Azure Container Apps is configured with `1 CPU`, `2Gi` memory, `min-replicas 0`, and `max-replicas 10`.
- This means the API can scale to zero when idle and scale out to 10 replicas during traffic spikes.
- The frontend CI/CD runs through GitHub Actions on pushes and pull requests to `main`.
- The frontend deploys from `./frontend` to Azure Static Web Apps, with production output in the `build` folder.

## Interview Answers

### 1. How did you containerize the backend?

**Situation:** The backend is a FastAPI ML inference service that needed to run consistently locally and in Azure.  
**Task:** I needed a reliable Docker image for the API and model files.  
**Action:** I used `python:3.11-slim`, installed only required system packages, cached dependencies by copying `requirements.txt` first, used `pip --no-cache-dir`, exposed port `8000`, and added a `/health` Docker healthcheck.  
**Result:** The same image can run locally with Docker and in Azure Container Apps with health monitoring built in.

### 2. How does the Azure deployment work?

**Situation:** I needed the backend deployed as a cloud-hosted API.  
**Task:** I wanted the deployment to be repeatable from a script.  
**Action:** The script creates `exoplanet-rg` in `eastus`, creates ACR `hamzaexoplanetacr1006`, builds `exoplanet-api:v1`, creates `exoplanet-env`, and deploys the Container App with external ingress on port `8000`.  
**Result:** The backend is available as a public HTTPS Azure Container Apps endpoint.

### 3. How does the backend scale?

**Situation:** ML inference traffic can be unpredictable.  
**Task:** I needed the backend to save cost when idle but handle spikes during use.  
**Action:** I configured Azure Container Apps with `min-replicas 0`, `max-replicas 10`, `1 CPU`, and `2Gi` memory per replica.  
**Result:** The API can scale to zero when unused and scale out up to 10 replicas when requests increase.

### 4. What CI/CD is implemented?

**Situation:** The frontend needed automatic deployment after code changes.  
**Task:** I needed a simple CI/CD workflow connected to Azure Static Web Apps.  
**Action:** GitHub Actions runs on pushes and pull requests to `main`, builds from `./frontend`, outputs to `build`, and deploys using `Azure/static-web-apps-deploy@v1` with Azure secrets.  
**Result:** Frontend changes can be deployed automatically through GitHub without manual portal steps.

## One-Line Closing

Overall, this project shows that I can package an application with Docker, deploy it to Azure, configure autoscaling, and use GitHub Actions for CI/CD in a practical production-style workflow.
