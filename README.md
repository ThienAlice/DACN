# Building and Securing CI/CD Pipelines: Applying DevSecOps for End-to-End Application Deployment

Welcome to the **CI/CD Pipeline Integration and Security** project! This project applies various popular tools to ensure that the Continuous Integration and Continuous Deployment (CI/CD) processes are automated, efficient, and secure.

## Contributor

- **Ngô Minh Thiên** - 21522623  
- **Nguyễn Đình Bảo Long** - 21522303  

## Table of Contents

- [Application Code](#application-code)
- [Jenkins Pipeline Code](#jenkins-pipeline-code)
- [Kubernetes Manifests Files](#kubernetes-manifests-files)
- [Tools Used](#tools-used)

## Workflow

![DACN](https://hackmd.io/_uploads/ryJMMqUkyg.png)

## Application Code

The `Application-Code` directory contains the source code for the application, including the frontend developed with **ReactJS** and the backend with **NodeJS**.


## Jenkins Pipeline Code

In the `Jenkins-Pipeline-Code` directory, you will find Jenkins pipeline scripts. These scripts automate the CI/CD process, ensuring smooth integration and deployment of your application.

---

## Kubernetes Manifests Files

The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on **AKS** (Azure Kubernetes Service). Customize these files to suit your project needs.

## Tools Used

In this project, we applied various popular tools to ensure the CI/CD processes are automated, efficient, and secure. The tools include:

| **Tool**                     | **Function**                                                                  |
|------------------------------|-------------------------------------------------------------------------------|
| **GitHub**                   | Manages source code and triggers the pipeline on changes.                     |
| **Jenkins**                  | CI tool responsible for integration, testing, and deployment.                 |
| **SonarQube**                | Performs static application security testing (SAST).                          |
| **Snyk**                     | Scans for vulnerabilities in third-party components (SCA).                    |
| **Maven**                    | Manages projects, compiles, tests, and builds applications.                   |
| **Nexus Repository**         | Stores artifacts after the build process.                                     |
| **Docker**                   | Creates and manages Docker images for application deployment.                  |
| **OWASP Zap**                | A tool for dynamic application security testing (DAST).                       |
| **Trivy**                    | Scans for security vulnerabilities in Docker images.                          |
| **Azure Container Registry**  | Stores Docker images.                                                         |
| **Kubernetes (K8s)**         | Manages and deploys application containers.                                   |
| **Argo CD**                  | Implements GitOps for Continuous Deployment, automating application updates on Kubernetes. |
| **Prometheus & Grafana**     | Monitors system performance and sets up alerts.                               |

---
