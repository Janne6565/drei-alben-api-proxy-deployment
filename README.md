# drei-alben-api-proxy-deployment

Kubernetes deployment manifests for drei-alben-api-proxy using ArgoCD.

## Components

- **PostgreSQL Database**: StatefulSet with persistent storage for push notification tokens
- **Backend API**: Spring Boot application for album data and push notifications
- **Ingress**: HTTP routing for external access

## Prerequisites

- Kubernetes cluster with ArgoCD installed
- Namespace: `drei-alben`

## Setup Instructions

### 1. Create Secrets

Before deploying, create the required secrets manually:

```bash
# PostgreSQL credentials
kubectl create secret generic postgres-secret \
  --namespace drei-alben \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=<YOUR_SECURE_PASSWORD> \
  --from-literal=POSTGRES_DB=drei_alben

# Backend API secrets
kubectl create secret generic drei-alben-api-proxy-secrets \
  --namespace drei-alben \
  --from-literal=DDFDB_API_KEY=<YOUR_DDFDB_API_KEY>
```

**Note**: The `postgres-secret.yaml` file in this repo is a template only. Do not commit actual passwords!

### 2. Deploy with ArgoCD

ArgoCD will automatically sync and deploy all manifests in the `k8s/` directory.

### 3. Verify Deployment

```bash
# Check PostgreSQL is running
kubectl get pods -n drei-alben -l app=postgres

# Check backend is running
kubectl get pods -n drei-alben -l app=drei-alben-proxy

# Check database connection
kubectl logs -n drei-alben -l app=drei-alben-proxy
```

## Database

- **Type**: PostgreSQL 16
- **Storage**: 5Gi PersistentVolume
- **Service**: `postgres.drei-alben.svc.cluster.local:5432`
- **Database Name**: `drei_alben`

### Migrations

Database schema is managed by Flyway. Migrations run automatically on application startup.

### Connecting to Database (for debugging)

```bash
kubectl port-forward -n drei-alben svc/postgres 5432:5432
psql -h localhost -U postgres -d drei_alben
```

## Push Notifications

The backend supports Expo Push Notifications for new album releases:

- Tokens stored in PostgreSQL
- Notifications sent when new albums detected (every 5 minutes)
- API endpoints: `/api/v1/push-tokens/*`

## Updating Deployment

1. Update manifests in this repository
2. Commit and push changes
3. ArgoCD will automatically sync and apply changes

For immediate sync:
```bash
argocd app sync drei-alben-api-proxy
```

## Troubleshooting

### Backend won't start

Check if PostgreSQL is ready:
```bash
kubectl get pods -n drei-alben -l app=postgres
kubectl logs -n drei-alben postgres-0
```

### Database connection errors

Verify secrets are created:
```bash
kubectl get secrets -n drei-alben
kubectl describe secret postgres-secret -n drei-alben
```
