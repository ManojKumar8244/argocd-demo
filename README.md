# GitOps Application Deployment using ArgoCD on Kubernetes

**Assignment Details:** Project 3.4[cite: 1]

## Problem Statement Overview
Manual deployments often lead to difficult rollbacks and inconsistent cluster configurations across environments[cite: 1]. This project implements a GitOps approach to solve these issues by ensuring the Kubernetes cluster state strictly matches the configurations stored in a Git repository.

## Solution Approach
I deployed an Nginx application using a GitOps workflow[cite: 1]. Kubernetes manifests defining the deployment and service were stored in this GitHub repository. ArgoCD was installed on the Kubernetes cluster and configured to monitor this repository. When changes are committed to the repository (such as scaling the Nginx deployment replicas), ArgoCD detects the drift and automatically synchronizes the updates to the cluster. 

## Dependencies and Tools
* Kubernetes Cluster (EKS or Local Cluster)[cite: 1]
* ArgoCD
* GitHub
* Docker Registry
* `kubectl` CLI

## Execution Steps
1. **Cluster Setup:** Provision a Kubernetes cluster and verify access using `kubectl`[cite: 1].
2. **Install ArgoCD:** Deploy the ArgoCD controller into the cluster.
3. **Configure Repository:** Clone this repository containing the Nginx manifests.
4. **Deploy Application:** Configure an ArgoCD Application resource to point to this Git repository and the target cluster namespace. 
5. **Sync and Verify:** Trigger the initial sync via the ArgoCD dashboard and verify the Nginx pods are running.
6. **Test Synchronization:** Modify the `replicas` count in the deployment manifest, commit, and push to Git to observe ArgoCD automatically applying the update.
