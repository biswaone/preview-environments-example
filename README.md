# Preview Environments Example

This repository demonstrates how to set up preview environments for your applications on a local Kubernetes cluster using Minikube, ArgoCD, GitHub Actions, and Helm.

## Project Overview

The project consists of a simple FastAPI application, a Dockerfile to containerize it, a Helm chart to deploy it, and the necessary CI/CD and GitOps configurations to automate the creation of preview environments.

## How it Works

1.  **Push to a branch**: A developer pushes code to a branch in the GitHub repository.
2.  **CI Pipeline**: A GitHub Actions workflow is triggered, which builds a Docker image and pushes it to GitHub Container Registry (GHCR). The image is tagged with the commit SHA.
3.  **Update Helm Values**: The workflow then updates the `values.yaml` file in the `helm` directory with the new image tag and commits the change to the repository.
4.  **Pull Request**: The developer creates a pull request and adds the `preview` label.
5.  **ArgoCD ApplicationSet**: An ArgoCD ApplicationSet is configured to watch for pull requests with the `preview` label. When it detects a new pull request, it automatically creates a new application in ArgoCD.
6.  **Preview Environment**: The ArgoCD application deploys the application to a new namespace in the Kubernetes cluster using the Helm chart from the pull request branch. The ingress is configured to be accessible at `pr-<branch-name>.<your-domain>`.

## Local Setup with Minikube

This section provides a step-by-step guide to setting up the preview environments on a local Minikube cluster.

### Prerequisites

*   [Minikube](https://minikube.sigs.k8s.io/docs/start/)
*   [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
*   [Helm](https://helm.sh/docs/intro/install/)

### 1. Start Minikube

Start a new Minikube cluster:

```bash
minikube start
```

### 2. Enable Ingress

Enable the ingress controller addon:

```bash
minikube addons enable ingress
```

### 3. Install ArgoCD

Install ArgoCD using the official Helm chart:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace
```

### 4. Get Minikube IP

Get the IP address of your Minikube cluster:

```bash
minikube ip
```

### 5. Update ApplicationSet

Update the `appset/appset.yml` file with the Minikube IP address. Replace `192-168-67-2` in the `ingress.host` value with the IP address from the previous step.

For example, if your Minikube IP is `192.168.49.2`, the `ingress.host` should be:
```yaml
value: pr-{{branch}}.192-168-49-2.nip.io
```


### 6. Apply the ApplicationSet

Apply the ApplicationSet to your cluster:

```bash
kubectl apply -f appset/appset.yml -n argocd
```

Now, when you create a pull request with the `preview` label, ArgoCD will automatically create a preview environment for you.

## A Note on `nip.io`

This project uses `nip.io` to provide a wildcard DNS for your local IP address. This allows you to access your preview environments using a domain name like `pr-my-branch.192-168-1-1.nip.io`, which resolves to `192.168.1.1`. This service is free to use and does not require any registration.


## CI/CD Pipeline

The CI/CD pipeline is defined in `.github/workflows/build-and-deploy.yml`. It consists of two jobs:

*   `build-push`: Builds the Docker image and pushes it to GHCR.
*   `deployment-job`: Updates the `values.yaml` file with the new image tag.

## Preview Environments

Preview environments are automatically created for each pull request with the `preview` label. The ApplicationSet in `appset/appset.yml` is responsible for this. It uses the `pullRequest` generator to create an application for each pull request. The application is deployed to a namespace named `preview-<pr-number>` and is accessible at `pr-<branch-name>.<your-domain>`.

## Acknowledgements

This project was inspired by the following resources:

*   **GitHub Repository**: [brandonphillips/preview-environments-example](https://github.com/brandonphillips/preview-environments-example)
*   **YouTube Video**: [GitOps for Preview Environments on Kubernetes](https://youtu.be/7ahiwZuiCBM?si=Lv_8JJj_3Rt8cRWN)
