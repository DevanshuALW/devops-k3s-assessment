# devops-k3s-assessment
This repository contains my submission for the Infrastructure DevOps Intern Assessment.  
The objective of this assessment was to set up and own a small production-like infrastructure, deploy an application on Kubernetes, debug an intentional failure, and explain operational decisions and tradeoffs.

devops-k3s-assessment/
├── apps/
│ └── deployment.yml
├── helm/
│ └── .gitkeep
├── k3s/
│ └── install.sh
├── terraform/
│ ├── main.tf
│ ├── outputs.tf
│ ├── variables.tf
│ └── versions.tf
├── .gitignore
└── README.md


Some directories (Helm and Terraform) are included as part of the base repository structure but were not actively used during this assessment, as the instructions focused on manual infrastructure ownership and Helm-based application deployment.

---

# Environment

- Cloud Provider: Hetzner
- Operating System: Ubuntu 24
- Access: SSH (root user)
- Container Runtime: Docker
- Kubernetes Distribution: Single-node k3s
- Package Manager: Helm v3+

The VM provided for this assessment was temporary and available only during the evaluation window.

---

# Infrastructure Setup

# Docker Installation
Docker was installed and verified to be running:

docker version
docker ps

# k3s Installation
curl -sfL https://get.k3s.io | sh -
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get nodes
kubectl get pods -A

# Helm Installation
Helm v3 was installed for application deployment:
helm version

# Application Deployment (Open WebUI)
helm repo add open-webui https://helm.openwebui.com/
helm repo update

kubectl create namespace openwebui

helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP \
  --dry-run

helm install webui open-webui/open-webui \
  --namespace openwebui \
  --set service.type=ClusterIP

# Deployment validation:
kubectl get all -n openwebui

# Debugging OIDC Failure

After enabling OIDC authentication, the application failed to start as expected.

The issue was identified by checking pod status in the openwebui namespace and inspecting the application logs. The logs showed that the failure occurred during OIDC initialization due to an invalid or unreachable issuer URL. Since the application performs strict validation during startup, it exited instead of running.

To get the application running again, the OIDC configuration was temporarily disabled and the application was redeployed. Once OIDC was disabled, the application started successfully. This fix works because it removes the failing authentication initialization step. In a real production setup, OIDC would only be enabled after the issuer, certificates, and client configuration are fully verified.

# Production Considerations

Single-node clusters introduce a single point of failure
Monitoring and alerting are required before production usage
Secrets should be managed securely and must never be committed to Git
Backups are required for any data that cannot be easily recreated
Infrastructure costs are minimized by avoiding over-provisioning early

# Validation Output
kubectl get nodes
kubectl get all -n openwebui

