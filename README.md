# 🚀 Migrating a Multi-Microservice Application and Implementing CI on Azure DevOps

[![Azure DevOps](https://img.shields.io/badge/Azure_DevOps-0078D7?style=flat-square&logo=azure-devops&logoColor=white)](https://azure.microsoft.com/en-us/services/devops/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)
[![.NET](https://img.shields.io/badge/.NET-512BD4?style=flat-square&logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)

## 📖 Overview

This repository documents the migration of a polyglot, multiple microservice application from GitHub to the Azure DevOps platform and the implementation of a comprehensive Continuous Integration (CI) system using Azure Pipelines.

This project is derived from the official Docker "example-voting-app".

### 🎯 Objective

As a DevOps Engineering team, our responsibility was to take the complete source code provided by the development team and implement robust CI pipelines on Azure DevOps. The key focus areas include:

1.  Migrating the source code from GitHub to Azure Repos.
2.  Containerizing three polyglot services (Python, Node.js, .NET).
3.  Writing service-specific Dockerfiles tailored for microservices.
4.  Implementing path-based CI triggers in Azure Pipelines.
5.  Setting up Self-Hosted Azure Pipeline Agents on Azure VMs.
6.  Automating build and push stages to an Azure Container Registry (ACR).

---

## 🏗️ Architecture

The application is a standard voting mechanism where users vote between two options, and the results are tallied and displayed in real-time. Although designed for educational purposes rather than production, it provides a robust scenario for CI/CD implementation.

### High-Level Service Flow

1.  **Voting App (Frontend):** Python-based web app. Takes user input.
2.  **Redis:** In-memory data store. Temporarily stores the votes.
3.  **Worker:** .NET-based background service. Consumes votes from Redis.
4.  **PostgreSQL:** Persistent database. Worker writes votes to DB.
5.  **Result App (Result Frontend):** Node.js-based web app. Displays tallied results from DB.

## 🎯 Key Project Objectives

- **Repository Migration:** Imported source code from GitHub into private **Azure Repos** (default branch: `main`).
- **Polyglot Containerization:** Analyzed and configured specific Dockerfiles for Python, Node.js, and .NET.
- **Self-Hosted Agent Optimization:** Built an Ubuntu Linux VM to act as a self-hosted Azure Pipeline agent, bypassing free-tier runner limitations.
- **Path-Based Triggers:** Configured isolated pipelines so modifying a single service folder only builds and pushes that specific microservice.
- **Artifact Management:** Automated image compilation and storage within **Azure Container Registry (ACR)**.

---

## ⚙️ CI Pipeline Configuration (YAML Example)

Each microservice utilizes an independent pipeline. Below is the optimized structure for the **Result Service** (`Azure-Pipelines-Result.yml`):

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - result/* # Trigger ONLY when files change inside the result directory

variables:
  dockerRegistryServiceConnection: "acr-service-connection"
  imageRepository: "result-app"
  containerRegistry: "yourregistry.azurecr.io"
  dockerfilePath: "$(Build.SourcesDirectory)/result/Dockerfile"

stages:
  - stage: Build
    displayName: "Build Stage"
    jobs:
      - job: BuildJob
        pool:
          name: "AzureAgent" # Points to the self-hosted VM agent
        steps:
          - task: Docker@2
            displayName: "Compile Image"
            inputs:
              command: "build"
              repository: $(imageRepository)
              dockerfile: $(dockerfilePath)
              containerRegistry: $(dockerRegistryServiceConnection)

  - stage: Push
    displayName: "Push Stage"
    dependsOn: Build
    jobs:
      - job: PushJob
        pool:
          name: "AzureAgent"
        steps:
          - task: Docker@2
            displayName: "Push to ACR"
            inputs:
              command: "push"
              repository: $(imageRepository)
              containerRegistry: $(dockerRegistryServiceConnection)
```

## 🚧 Challenges & Solutions

During the **.NET Worker** build, multi-architecture variables (`Target Platform`/`Target Arch`) caused compilation failures on the self-hosted runner.

**DevOps Resolution:** Modified the worker's Dockerfile to strip out multi-arch parameters and explicitly locked the build targets to match the runner host's Linux environment. This stabilized the pipeline execution while maintaining standard image integrity.

---

## ⏭️ Next Steps

With 100% of the Continuous Integration framework successfully configured and pushing verified builds to ACR, next project will focus on **Continuous Delivery (CD)**, pulling these images from the registry and deploying them to target environments.
