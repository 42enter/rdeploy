# rdeploy

A CLI tool that makes it easy to build and deploy applications to Kubernetes. It wraps common tools (helm, kubectl, gcloud, az) into simple commands configured via a project-level YAML file.

## Installation

```bash
pip install rdeploy
```

## Quick Start

1. Create an `rdeploy.yaml` in your project root (see [Configuration](#configuration))
2. Run commands:

```bash
# Set cluster context
rdeploy set-context staging

# Build and push docker image
rdeploy cloudbuild staging v1.2.3

# Deploy to kubernetes
rdeploy upgrade staging v1.2.3

# Create a new release tag
rdeploy git-release patch
```

## Commands

| Command | Description |
|---------|-------------|
| `set-project` | Set the active cloud project (GCP/Azure) |
| `set-cluster` | Set the active Kubernetes cluster |
| `set-context` | Switch cluster and namespace context |
| `activate` | Fetch and set project, cluster, and namespace |
| `install` | Install a Helm chart deployment |
| `upgrade` | Upgrade a Helm deployment to a new image tag |
| `helm` | Run arbitrary Helm commands |
| `helm-setup` | Download and configure a local Helm binary |
| `git-release` | Bump version and push a git tag |
| `next-version` | Show what the next version number would be |
| `latest-version` | Show the current latest version from git tags |
| `build` | Build and push a Docker image |
| `cloudbuild` | Build a Docker image via cloud build (GCP/Azure) |
| `live-image` | Display the currently deployed image and tag |
| `shell` | Exec into a management container |
| `manage` | Run a Django management command in-cluster |
| `create-namespace` | Create a Kubernetes namespace |
| `upload-secrets` | Upload secrets from an env file to Kubernetes |
| `decode-secret` | Print decoded values of a Kubernetes secret |
| `upload-static` | Upload static files to a cloud bucket |
| `create-bucket` | Create a private cloud storage bucket |
| `create-public-bucket` | Create a public cloud storage bucket |
| `create-volume` | Create a persistent disk |
| `compose` | Wrapper for docker-compose |

## Configuration

rdeploy is configured via an `rdeploy.yaml` file in your project root. The file supports versioned schemas (v1, v2, v3).

### Version 3 (recommended)

```yaml
version: '3'
configs:
  staging:
    project_name: my-service
    namespace: staging
    kube_context: gke_my-project_us-central1_cluster-1
    docker_image: gcr.io/my-project/my-service:latest
    cloud_provider:
      name: gcp
      project: my-project
      kube_cluster: cluster-1
      region: us-central1
      helm_registry: us-central1-docker.pkg.dev
    helm_values_path: ./etc/helm/staging/values.yaml
    helm_chart: rehive-service
    helm_chart_version: 0.2.0
    helm_version: 3.14.0
    use_system_helm: true

  production:
    project_name: my-service
    namespace: production
    kube_context: gke_my-project_us-central1_cluster-1
    docker_image: gcr.io/my-project/my-service:latest
    cloud_provider:
      name: gcp
      project: my-project
      kube_cluster: cluster-1
      region: us-central1
      helm_registry: us-central1-docker.pkg.dev
    helm_values_path: ./etc/helm/production/values.yaml
    helm_chart: rehive-service
    helm_chart_version: 0.2.0
    helm_version: 3.14.0
    use_system_helm: true
```

### Azure Example

```yaml
version: '2'
configs:
  production:
    project_name: my-service
    namespace: production
    docker_image: myregistry.azurecr.io/my-service
    cloud_provider:
      name: azure
      subscription_id: 57800aa9-e115-4ede-9b70-0e1ce97294f6
      resource_group: my-resource-group
      region: westeurope
      kube_cluster: my-cluster
      container_registry: myregistry
    helm_values_path: ./etc/helm/production/values.yaml
    helm_chart: rehive/rehive-service
    helm_chart_version: 0.1.38
    helm_version: 3.14.0
    use_system_helm: true
```

### Configuration Reference

| Field | Description |
|-------|-------------|
| `version` | Config schema version (`1`, `2`, or `3`) |
| `project_name` | Name of the Helm release and Kubernetes deployment |
| `namespace` | Kubernetes namespace |
| `kube_context` | (v3) Direct kubectl context name |
| `docker_image` | Docker image path |
| `cloud_provider.name` | Cloud provider (`gcp` or `azure`) |
| `cloud_provider.project` | GCP project ID |
| `cloud_provider.kube_cluster` | Kubernetes cluster name |
| `cloud_provider.region` | Cloud region |
| `cloud_provider.zone` | Cloud zone (GCP, alternative to region) |
| `cloud_provider.helm_registry` | OCI Helm registry (GCP Artifact Registry) |
| `cloud_provider.container_registry` | Azure container registry name |
| `helm_values_path` | Path to Helm values file |
| `helm_chart` | Helm chart name or OCI reference |
| `helm_chart_version` | Helm chart version |
| `helm_version` | Helm binary version (for local installs) |
| `use_system_helm` | Use system Helm binary (default: `true`) |

## Versioning

rdeploy uses semantic versioning for releases. The `git-release` command automates tagging:

```bash
# Patch release: 1.2.3 → 1.2.4
rdeploy git-release patch

# Minor release: 1.2.3 → 1.3.0
rdeploy git-release minor

# Major release: 1.2.3 → 2.0.0
rdeploy git-release major

# Pre-release: 1.2.3 → 1.2.4-rc.1
rdeploy git-release pre-patch

# Skip confirmation prompt
rdeploy git-release patch --force
```

## Development

```bash
# Clone and install in development mode
git clone https://github.com/rehive/rdeploy.git
cd rdeploy
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"

# Run tests
pytest

# Run with coverage
coverage run -m pytest
coverage report
```

## Requirements

- Python 3.9+
- kubectl configured with cluster access
- Cloud CLI tools as needed: `gcloud` (GCP), `az` (Azure)
- Helm 3.x

## Publishing to PyPI

```bash
rm -rf dist
python -m build
twine upload dist/*
```
