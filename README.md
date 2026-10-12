# gitops-playground

## Structure

```text
.
├── apps/       # Application workloads deployed to Kubernetes
├── bootstrap/  # Resources required to bootstrap Argo CD
├── clusters/   # Defines what is deployed to each Kubernetes cluster
├── docs/       # Project documentation and architecture notes
└── platform/   # Shared infrastructure resources
```

## Local bootstrap

See the [bootstrap guide](docs/bootstrap.md) to set up the local cluster.
