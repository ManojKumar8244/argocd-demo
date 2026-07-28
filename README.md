# GitOps Application Deployment using ArgoCD on Google Kubernetes Engine (GKE)

## Objective

Deploy a Kubernetes application using the GitOps approach. Instead of
manually applying Kubernetes YAML files after every change, ArgoCD
continuously monitors a GitHub repository and automatically synchronizes
the Kubernetes cluster whenever changes are pushed.

------------------------------------------------------------------------

# Architecture

``` text
Developer
    │
    │ git push
    ▼
GitHub Repository
(Source Code + Kubernetes Manifests)
    │
    │ ArgoCD watches repository
    ▼
ArgoCD (GitOps Controller)
    │
    │ Detects new commit
    ▼
Kubernetes Manifests
(Deployment, Service, ConfigMap...)
    │
    │ Applies manifests
    ▼
Google Kubernetes Engine (GKE)
(Kubernetes Cluster)
    │
    ├── Deployment
    ├── ReplicaSet
    ├── Pods
    ├── Service
    └── Ingress (Optional)
    │
    ▼
Application Running
```

------------------------------------------------------------------------

# Workflow

1.  Create a GKE cluster.
2.  Install ArgoCD into the cluster.
3.  Create a GitHub repository containing Kubernetes manifests.
4.  Configure an ArgoCD Application pointing to the GitHub repository.
5.  ArgoCD monitors the repository.
6.  Push manifest changes to GitHub.
7.  ArgoCD detects the commit and synchronizes the cluster.
8.  Verify the application and test automatic synchronization.

------------------------------------------------------------------------

# Prerequisites

-   Google Cloud account
-   GKE cluster
-   kubectl
-   gcloud CLI
-   Git
-   GitHub repository
-   Internet connection

------------------------------------------------------------------------

# Step 1 -- Create a GKE Cluster

Create the Kubernetes cluster from Google Cloud Console or CLI.

Example:

``` bash
gcloud container clusters create gitops-cluster \
    --zone us-central1-a \
    --num-nodes 2
```

Connect kubectl:

``` bash
gcloud container clusters get-credentials gitops-cluster --zone us-central1-a
```

Verify:

``` bash
kubectl get nodes
```

Expected:

    All nodes should be Ready.

------------------------------------------------------------------------

# Step 2 -- Install ArgoCD

Create the namespace.

``` bash
kubectl create namespace argocd
```

Install ArgoCD.

``` bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify installation.

``` bash
kubectl get pods -n argocd
```

Expected:

    All ArgoCD pods should be Running.

------------------------------------------------------------------------

# Step 3 -- Access ArgoCD Dashboard

Forward the ArgoCD service.

``` bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

    https://localhost:8080

Get the initial admin password.

``` bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 --decode
```

Username:

    admin

------------------------------------------------------------------------

# Step 4 -- Prepare GitHub Repository

Repository example:

``` text
gitops-demo/
│
├── deployment.yaml
├── service.yaml
└── README.md
```

Example Deployment:

``` yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: kubeserve-demo1
  namespace: default

spec:
  replicas: 4

  selector:
    matchLabels:
      app: kubeserve

  template:
    metadata:
      labels:
        app: kubeserve

    spec:
      containers:
      - name: app
        image: leaddevops/kubeserve:v2
```

Example Service:

``` yaml
apiVersion: v1
kind: Service

metadata:
  name: kubeserve-svc
  namespace: default

spec:
  type: NodePort

  selector:
    app: kubeserve

  ports:
  - port: 80
    targetPort: 80
```

Commit and push:

``` bash
git add .
git commit -m "Initial Kubernetes manifests"
git push origin main
```

------------------------------------------------------------------------

# Step 5 -- Create an ArgoCD Application

Using the ArgoCD UI:

-   Application Name: kubeserve
-   Project: default
-   Sync Policy: Manual (or Automatic)
-   Repository URL: Your GitHub repository
-   Revision: main
-   Path: /
-   Cluster: https://kubernetes.default.svc
-   Namespace: default

Click **Create**.

------------------------------------------------------------------------

# Step 6 -- Synchronize the Application

If using Manual Sync:

-   Open the application.
-   Click **SYNC**.
-   Confirm synchronization.

ArgoCD reads the repository and applies the Kubernetes manifests to the
GKE cluster.

------------------------------------------------------------------------

# Step 7 -- Verify Deployment

Check the deployment:

``` bash
kubectl get deployments
```

Check Pods:

``` bash
kubectl get pods
```

Check Service:

``` bash
kubectl get svc
```

Describe deployment if required:

``` bash
kubectl describe deployment kubeserve-demo1
```

------------------------------------------------------------------------

# Step 8 -- Test GitOps Synchronization

Edit deployment.yaml.

Example:

``` yaml
replicas: 4
```

Change to:

``` yaml
replicas: 6
```

Commit and push.

``` bash
git add .
git commit -m "Scale application"
git push origin main
```

ArgoCD automatically detects the Git commit.

Verify:

``` bash
kubectl get pods
```

Expected:

    6 Pods running.

------------------------------------------------------------------------

# Expected Outcome

-   GKE cluster is running.
-   ArgoCD is installed successfully.
-   GitHub stores Kubernetes manifests.
-   ArgoCD continuously monitors GitHub.
-   Every Git commit automatically updates the Kubernetes cluster.
-   Application stays synchronized with Git.

------------------------------------------------------------------------

# Author

**Manoj Kumar Nagamulla**

- GitHub: https://github.com/ManojKumar8244
