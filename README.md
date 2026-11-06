# AWS ECR to Kubernetes (k3s) Deployment

## Project Overview

This project demonstrates building a Docker container image on an Arch Linux workstation, publishing it to a private Amazon Elastic Container Registry (ECR), and deploying it to a Kubernetes (k3s) cluster running on a Raspberry Pi 5.

The project also implements least-privilege IAM access and automated renewal of the Kubernetes ECR image pull credentials using a systemd user timer.

## Architecture

```text
Arch Linux Workstation (bruno)
          |
          | Docker Buildx (linux/arm64)
          v
    Container Image
          |
          | Docker Push
          v
    Amazon ECR (Private)
          |
          | Authenticated Image Pull
          v
 Raspberry Pi 5 — Kubernetes (k3s)
          |
          v
    NGINX Deployment
          |
          v
    ClusterIP Service
          |
          v
    Browser (kubectl port-forward)
```

## Technologies Used

- Amazon Web Services (AWS)
- Amazon Elastic Container Registry (ECR)
- AWS Identity and Access Management (IAM)
- Docker and Docker Buildx
- Kubernetes (k3s)
- Raspberry Pi 5 (ARM64)
- NGINX
- Linux and Bash
- systemd user services and timers

## Kubernetes Configuration

The application runs in the `aws-ecr-demo` namespace.

The Deployment uses one replica of NGINX and references the private Amazon ECR image using the `v1` tag.

Kubernetes authenticates to ECR through an image pull secret named `ecr-pull-secret`.

The application is exposed internally through a ClusterIP Service on port 80.

## Security

A dedicated IAM user, `k3s-ecr-pull`, was created with a custom least-privilege policy named `K3sECRPullOnly`.

Permissions are limited to retrieving an ECR authorization token and downloading image data from the project repository.

AWS credentials and Kubernetes image pull secrets are not stored in this Git repository.

## Automated ECR Authentication

Amazon ECR authorization tokens expire after 12 hours.

A Bash script running on the Arch Linux workstation refreshes the Kubernetes image pull secret using a restricted AWS CLI profile.

A systemd user timer schedules renewal every six hours.

systemd lingering is enabled so the user manager can operate without an active login session.

This automation depends on the workstation being powered on, AWS connectivity, valid IAM credentials, and connectivity to the Kubernetes API.

## Validation

The following checks were completed:

- Built an ARM64-compatible NGINX container image
- Pushed the image to a private Amazon ECR repository
- Verified the ECR image digest against the running Kubernetes container
- Deployed the application successfully to the Raspberry Pi 5
- Confirmed the pod was Running and Ready
- Verified the NGINX web page through Kubernetes port forwarding
- Manually tested ECR authentication renewal
- Enabled the scheduled systemd renewal timer

## Repository Structure

```text
aws-ecr-k3s-demo/
├── Dockerfile
├── index.html
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── .gitignore
└── README.md
```

## Lessons Learned

This project provided hands-on experience with container architecture compatibility, private container registries, IAM least-privilege permissions, Kubernetes image pull authentication, resource configuration, and Linux automation.

It also demonstrated how AWS cloud services can integrate with a self-hosted Kubernetes environment.