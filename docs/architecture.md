# Architecture

## High-level flow

```
Developer pushes to `main`
          │
          ▼
   GitHub Actions triggered
          │
          ├──► Backend workflow                ├──► Frontend workflow
          │      │                              │      │
          │      ▼                              │      ▼
          │  Configure AWS credentials           │  Configure AWS credentials
          │  (via GitHub Actions secrets)         │  (via GitHub Actions secrets)
          │      │                              │      │
          │      ▼                              │      ▼
          │  Login to Amazon ECR                  │  Login to Amazon ECR
          │      │                              │      │
          │      ▼                              │      ▼
          │  Docker build                         │  Docker build
          │      │                              │      │
          │      ▼                              │      ▼
          │  Tag image with run number            │  Tag image with run number
          │      │                              │      │
          │      ▼                              │      ▼
          └──► Push to ECR (netflix_backend)     └──► Push to ECR (netflix_frontend)
```

## Components

### Backend (Spring Boot + MongoDB)
- REST API built with Spring Boot
- Connects to a MongoDB Atlas cluster for persistent storage of content/movie data
- Containerized with a Dockerfile
- Database connection string is supplied via `application.properties` at build/runtime —
  never committed to source control (see `application.properties.example`)

### Frontend (React)
- React-based UI for browsing content, consuming the backend REST API
- Containerized with its own Dockerfile

### CI/CD pipeline (GitHub Actions)
Each service (`backend/`, `frontend/`) has its own workflow that runs on every push to
`main`:

1. **Checkout** — pulls the latest code
2. **AWS authentication** — uses `aws-actions/configure-aws-credentials`, with the
   access key, secret key, and region all pulled from GitHub Actions secrets (never
   hardcoded in the workflow file)
3. **ECR login** — authenticates Docker to the Amazon ECR registry
4. **Docker build** — builds the service's container image
5. **Tag & push** — tags the image using `${GITHUB_RUN_NUMBER}` (so every pipeline run
   produces a uniquely versioned image) and pushes it to ECR

This means every successful push to `main` results in a new, traceable, versioned
container image in ECR — ready to be picked up by a deployment step (e.g. updating an
ECS service or a Kubernetes Deployment).

## Security considerations
- AWS credentials are stored as encrypted GitHub Actions secrets, not in the codebase
- Database credentials are excluded from version control entirely
- Image tags are immutable per build (tied to the GitHub Actions run number), making
  rollbacks straightforward — redeploy a previous tag if needed
