# Netflix-Style Full-Stack App — Containerized & Deployed to AWS ECR

A full-stack movie/content browsing application (Spring Boot backend, MongoDB database,
React-based frontend) containerized with Docker and deployed via an automated CI/CD
pipeline to Amazon ECR.

## Overview

This project takes a complete full-stack application and automates its build and deployment:

- **Backend**: Spring Boot REST API, backed by MongoDB Atlas
- **Frontend**: React-based UI for browsing content
- **Containerization**: Both services are containerized with Docker
- **CI/CD**: GitHub Actions pipeline triggers on push to `main`, builds Docker images,
  authenticates to AWS, and pushes images to Amazon ECR
- **Registry**: Amazon ECR stores versioned image tags (tagged by GitHub Actions run number)

## Tech stack

| Layer | Tools |
|---|---|
| Backend | Java, Spring Boot |
| Database | MongoDB (Atlas) |
| Frontend | React |
| Containerization | Docker |
| CI/CD | GitHub Actions |
| Registry | Amazon ECR |
| Cloud auth | AWS IAM (via GitHub Actions secrets) |

## Repo structure

```
.
├── backend/            # Spring Boot application source + Dockerfile + CI/CD workflow
├── frontend/           # React application source + Dockerfile + CI/CD workflow
├── diagrams/           # Architecture diagram
└── docs/
    └── architecture.md   # Detailed architecture and pipeline write-up
```

## How it was built

1. Built the backend as a Spring Boot REST API connected to a MongoDB Atlas database.
2. Built the frontend as a React application consuming the backend API.
3. Containerized both services with individual Dockerfiles.
4. Set up a GitHub Actions workflow for each service that:
   - Checks out the repository
   - Authenticates to AWS using credentials stored in GitHub Actions secrets
   - Logs in to Amazon ECR
   - Builds the Docker image
   - Tags the image with the GitHub Actions run number
   - Pushes the image to ECR
5. Each push to `main` automatically produces a new, versioned container image ready
   for deployment.

See [docs/architecture.md](docs/architecture.md) for the full breakdown.

## Configuration and secrets

This repo intentionally does **not** include real credentials or environment values.

- `backend/application.properties.example` shows the required MongoDB connection
  format without real values — copy to `application.properties` and fill in your own.
- AWS credentials in the CI/CD workflows are referenced via GitHub Actions secrets
  (`${{ secrets.AWS_ACCESS_KEY_ID }}`, etc.) — never hardcoded.
- ECR repository URIs use a placeholder AWS account ID; replace `<your-account-id>`
  with your own when adapting this pipeline.

## What I'd improve next

- Add automated tests to the pipeline before the build/push step
- Use OIDC-based AWS authentication instead of long-lived access key secrets
- Add a deployment step (e.g. to ECS or EKS) after the image push
- Remove the debug `aws sts get-caller-identity` step before calling the pipeline production-ready
