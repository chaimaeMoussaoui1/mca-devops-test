
## Build steps

### Prerequisites
- Docker Desktop
- minikube
- kubectl

### Build the images

```shell
docker build -t backend:local ./backend
docker build -t frontend:local ./frontend
```

### Load the images into minikube

```shell
minikube image load backend:local
minikube image load frontend:local
```

## Deployment steps

### 1. Start the cluster

```shell
minikube start
```

### 2. Deploy the full stack

```shell
kubectl apply -f manifests/
```

This creates:
- PostgreSQL Deployment + Service
- Backend Deployment + Service (Spring Boot, connects to PostgreSQL)
- Frontend Deployment + Service (Angular, served by nginx with a reverse proxy to the backend on `/api/`)

### 3. Verify pods are running

```shell
kubectl get pods
```

### 4. Access the frontend

```shell
kubectl port-forward svc/frontend 8081:80
```

Open `http://127.0.0.1:8081` in a browser.



## Notes

- Following the "keep it simple" guideline, credentials are set directly as plain environment variables in the manifests (no ConfigMap/Secret abstraction).

## Troubleshooting encountered (with AI assistance)

- **Backend/frontend not reachable via `localhost`**: root cause was `localhost` resolving to IPv6 (`::1`) while `kubectl port-forward` only listens on `127.0.0.1`. Fixed by using `127.0.0.1` explicitly.
- **Frontend could not reach the backend API** (`404` on `/api/users`): the Angular production build calls the API via a relative path (`/api/`), so nginx needed a reverse proxy rule to forward `/api/` requests to the backend Service. Added `frontend/nginx.conf` and updated the Dockerfile to include it.
## Bonus: CI Pipeline (Jenkins)

Jenkins was deployed via Helm following the instructions in `jenkins/README.md`, with a `Jenkinsfile` using a `buildah` agent to build both images. The pipeline successfully authenticates with GitHub and checks out the repository. Due to local resource constraints (limited CPU/memory on the minikube VM), the Jenkins pod experienced intermittent instability preventing a fully stable end-to-end run — see `Jenkinsfile` for the pipeline definition.