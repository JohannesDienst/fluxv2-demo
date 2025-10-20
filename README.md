# Flux v2 Demo Repository

This repository demonstrates a GitOps setup using [Flux v2](https://fluxcd.io/) for managing Kubernetes infrastructure and applications. It includes multi-environment deployments (staging and production) of the [podinfo](https://github.com/stefanprodan/podinfo) application and the installation of the ingress-nginx controller via Helm.

## Repository Structure

```
├── apps/
│   ├── kustomization.yaml
│   └── podinfo/
│       ├── overlays/
│       │   ├── base/
│       │   ├── production/
│       │   └── staging/
│       └── sources/
│           └── flux-system/
└── clusters/
    └── my-cluster/
        ├── kustomization.yaml
        ├── flux-system/
        ├── infrastructure/
        ├── namespaces/
        └── rbac/
```

### Key Directories

- **apps/**:
    - Application definitions and Kustomizations for different environments
    - Cluster has two namespaces: *staging* and *production*
    - Every namespace is isolated through *Service Account* which can only  deploy resources to their own namespace
    - sources contains resources both namespaces need
- **clusters/my-cluster/**:
    - *flux-system* contains cluster config and Service Accounts 
    - *infrastructure*: ingress-nginx controller
    - *namespaces*: Namespace definitions
    - *RBAC*: Rolebindings for Service Accounts

---

## Application Deployment

### Podinfo

- **Source**: [stefanprodan/podinfo](https://github.com/stefanprodan/podinfo)
- **Environments**: `staging` and `production` overlays, each with its own Kustomization and Ingress.
- **Automated Scaling**: HPA patches set `minReplicas: 2` for both environments.

### Ingress-NGINX

- **Source**: [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)
- **Deployed via**: HelmRelease, using a GitRepository source and custom values.

---

## Namespaces

- `ingress-nginx`
- `staging`
- `production`

Each namespace is labeled for pod security and workspace separation.

---

## Getting Started

```sh
# 1. Bootstrap Flux
flux bootstrap github \
  --owner=<github-username> \
  --repository=<repo-name> \
  --branch=main \
  --path=clusters/my-cluster

# 2. Check Flux System Status
flux get all -A

# 3. Reconcile Resources
flux reconcile kustomization flux-system --with-source

# Reconcile a specific Kustomization (e.g., podinfo production):

flux reconcile kustomization podinfo --namespace=flux-system

# 4. View Application Status
flux get kustomizations -A
flux get helmreleases -A

# 5. Troubleshooting
kubectl -n flux-system logs deployment/kustomize-controller
kubectl -n flux-system logs deployment/helm-controller

# Check resource status:
kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A
kubectl get gitrepositories.source.toolkit.fluxcd.io -A
```

## Customization
* Add new applications: Place new app manifests under apps/ and reference them in the appropriate Kustomization.

* Add new clusters: Create a new directory under clusters/ and replicate the structure.
    Update sources: Edit the GitRepository or HelmRepository resources as needed.

## References
* [Flux Documentation](https://fluxcd.io/flux/)
* [Kustomize Documentation](https://kubectl.docs.kubernetes.io/guides/)
