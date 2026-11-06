# Architecture — AWS ECR to Kubernetes (k3s)

## 1. Overview

This project integrates AWS container registry services with a self-hosted Kubernetes cluster running on a Raspberry Pi 5.

The architecture separates image building, image storage, and application execution across three environments.

## 2. Components

| Component | Purpose |
|---|---|
| Arch Linux workstation (`bruno`) | Builds and pushes ARM64 container images |
| Docker Buildx | Builds images compatible with the Raspberry Pi |
| Amazon ECR | Privately stores container images |
| AWS IAM | Restricts access to the private registry |
| Raspberry Pi 5 (`raspi`) | Hosts the Kubernetes k3s cluster |
| Kubernetes Deployment | Manages the NGINX application |
| Kubernetes Service | Provides internal network access |
| systemd timer | Schedules ECR credential renewal |

## 3. Image Build and Deployment Flow

1. Build the NGINX container image using Docker Buildx with the `linux/arm64` platform.
2. Authenticate Docker to Amazon ECR using AWS CLI credentials.
3. Push the image tagged `v1` to the private ECR repository.
4. Configure a Kubernetes image pull secret for ECR authentication.
5. Apply the Kubernetes Deployment and Service manifests.
6. Kubernetes retrieves the image from ECR and starts the NGINX container on the Raspberry Pi.
7. Access the application through `kubectl port-forward`.

## 4. Authentication Architecture

A dedicated IAM user named `k3s-ecr-pull` provides restricted access to the private ECR repository.

The IAM policy permits obtaining an ECR authorization token and retrieving container images from the project repository.

AWS credentials remain on the workstation and are not committed to GitHub.

The Kubernetes image pull secret stores temporary registry authentication information.

## 5. Automated Credential Renewal

Amazon ECR authorization tokens are valid for 12 hours.

A Bash script on `bruno` obtains a fresh ECR token using the restricted AWS CLI profile and updates the Kubernetes image pull secret.

A systemd user timer schedules the script every six hours.

The systemd user manager has lingering enabled to support scheduled execution when no interactive user session is active.

The renewal mechanism requires the workstation to remain operational and able to reach both AWS and the Kubernetes API.

## 6. Networking

The NGINX application is exposed through a Kubernetes ClusterIP Service on port 80.

ClusterIP limits direct Service access to the cluster network.

For local browser testing, Kubernetes port forwarding makes the application accessible through `localhost:8080` on the workstation.

## 7. Operational Considerations

- Monitor the systemd timer and service logs for renewal failures.
- Keep AWS IAM access keys protected and rotate them periodically.
- Maintain network connectivity between the workstation, AWS, and the Kubernetes cluster.
- Verify image pull authentication when restarting or replacing pods.
- Avoid committing credentials, Kubernetes Secrets, or ECR tokens to source control.

## 8. Potential Improvements

Future enhancements could include GitHub Actions for image builds, automated image scanning, GitOps-based deployments, and additional monitoring for credential renewal failures.