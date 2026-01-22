# Cloud Sample Deployment

Infrastructure repository for Kubernetes deployment manifests.

## Structure

```
cloud-sample-deployment/
├── .github/workflows/
│   └── deploy.yml          # Deploy to GKE workflow
├── kustomize/
│   ├── base/               # Base K8s manifests
│   │   ├── gateway/        # API Gateway
│   │   ├── task/           # Task Service
│   │   ├── user/           # User Service
│   │   ├── frontend/       # React Frontend
│   │   ├── postgresql/     # PostgreSQL DB
│   │   └── mongodb/        # MongoDB
│   └── kustomization.yml   # Image versions
└── scripts/
    ├── create-secrets.sh   # Create K8s secrets
    └── deploy.sh           # Manual deployment
```

## How It Works

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  cloud (app repo)                    cloud-sample-deployment (this repo)   │
│                                                                             │
│  Push code ──► Build images ──► repository_dispatch ──► Deploy to GKE      │
│                                                                             │
│  [build-and-push.yml]                               [deploy.yml]           │
└─────────────────────────────────────────────────────────────────────────────┘
```

## GitHub Secrets Required

| Secret | Description |
|--------|-------------|
| `GKE_PROJECT` | Google Cloud Project ID |
| `GKE_CLUSTER` | GKE Cluster name |
| `GKE_ZONE` | GKE Zone (e.g., `europe-west1-b`) |
| `GKE_SA_KEY` | Service Account JSON key |
| `POSTGRES_PASSWORD` | PostgreSQL password |
| `MONGO_PASSWORD` | MongoDB password |
| `IMAGE_OWNER` | (Optional) GitHub username for images |

## App Repository Setup

The app repository needs:

| Secret | Description |
|--------|-------------|
| `DEPLOY_REPO_TOKEN` | Personal Access Token with `repo` scope |

### Creating the PAT

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token with `repo` scope
3. Add as `DEPLOY_REPO_TOKEN` secret in the app repository

## Manual Deployment

```bash
# Trigger deployment via GitHub Actions UI (workflow_dispatch)
# Or use gh CLI:
gh workflow run deploy.yml -f services=all -f image_tag=latest
```

## Local Deployment

```bash
# Connect to cluster
gcloud container clusters get-credentials CLUSTER_NAME --zone ZONE

# Create secrets
./scripts/create-secrets.sh

# Deploy
kubectl apply -k kustomize/
```
