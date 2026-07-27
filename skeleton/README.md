# ${{ values.component_id }}

${{ values.description }}

## Overview

This application demonstrates a complete CI/CD pipeline using:

- **OpenShift Dev Spaces**: Cloud-based development environment
- **Tekton Pipelines**: Cloud-native CI/CD automation
- **ArgoCD (OpenShift GitOps)**: GitOps continuous delivery
- **Docker Registry**: Container image storage

## Architecture

```
Dev Spaces → Git Push → Tekton Pipeline → Container Build → GitOps Sync → Deployment
```

## Getting Started

### Prerequisites

- OpenShift cluster with the following operators installed:
  - Red Hat OpenShift Dev Spaces
  - Red Hat OpenShift Pipelines
  - Red Hat OpenShift GitOps
- GitHub account with OAuth configured
- Container registry credentials (Docker Hub, Quay.io, etc.)

### Development Workflow

1. **Open in Dev Spaces**:
   ```
   https://devspaces${{ values.cluster }}/#https://${{ values.destination.host }}/${{ values.destination.owner }}/${{ values.destination.repo }}
   ```

2. **Make changes** in the browser-based VS Code environment

3. **Commit and push** - This automatically triggers:
   - Tekton pipeline clones the repository
   - Builds the Go binary
   - Creates container image with Buildah
   - Updates the deployment manifest with new image tag
   - Pushes manifest changes back to Git

4. **ArgoCD syncs** the changes to the cluster automatically

### Pipeline Details

The Tekton pipeline (`.tekton/push.yaml`) performs these tasks:

1. **clone-repo**: Clone the Git repository
2. **build-go**: Build the Go application
3. **build-image**: Create container image using Buildah
4. **update-deployment**: Update deployment manifest with new image tag

### ArgoCD Application

- **Name**: `${{ values.component_id }}`
- **Namespace**: `${{ values.namespace }}`
- **Sync Policy**: Automatic (with self-heal)

### Accessing the Application

Once deployed, access via:
```
https://${{ values.component_id }}-${{ values.namespace }}${{ values.cluster }}
```

## Required Secrets

Create these secrets in the `${{ values.namespace }}` namespace:

### Git Credentials
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: git-credentials
  namespace: ${{ values.namespace }}
  annotations:
    tekton.dev/git-0: https://${{ values.destination.host }}
type: kubernetes.io/basic-auth
stringData:
  username: <your-username>
  password: <your-token>
```

### Registry Credentials
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-credentials
  namespace: ${{ values.namespace }}
  annotations:
    tekton.dev/docker-0: https://${{ values.image_registry }}
type: kubernetes.io/basic-auth
stringData:
  username: <registry-username>
  password: <registry-password>
```

### Docker Config Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
  namespace: ${{ values.namespace }}
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

### Service Account
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-sa
  namespace: ${{ values.namespace }}
secrets:
  - name: regcred
  - name: git-credentials
imagePullSecrets:
  - name: registry-credentials
```

Then grant SCC:
```bash
oc adm policy add-scc-to-user pipelines-scc -z pipeline-sa -n ${{ values.namespace }}
```

## Monitoring

- **Tekton Pipelines**: https://console-openshift-console${{ values.cluster }}/dev-pipelines/ns/${{ values.namespace }}
- **ArgoCD Applications**: https://openshift-gitops-server-${{ values.argocd_namespace }}${{ values.cluster }}/applications/${{ values.component_id }}
- **Developer Hub**: Check the component page for integrated views

## References

- [Build a CI/CD Pipeline with OpenShift Dev Spaces and GitOps](https://developers.redhat.com/articles/2026/02/16/build-cicd-pipeline-openshift-dev-spaces-and-gitops)
- [Red Hat Developer Hub Documentation](https://access.redhat.com/documentation/en-us/red_hat_developer_hub)
- [OpenShift Pipelines Documentation](https://docs.openshift.com/pipelines/latest/about/understanding-openshift-pipelines.html)
- [OpenShift GitOps Documentation](https://docs.openshift.com/gitops/latest/understanding_openshift_gitops/about-redhat-openshift-gitops.html)
