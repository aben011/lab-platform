# lab-platform

Platform-level infra and cluster tooling.

## Bootstrapping ArgoCD

```bash
kubectl apply --server-side --force-conflicts --kustomize='argocd-controller'
```

## Gateway Secret via cert-manager / DuckDNS

Add DuckDNS API token to `secrets/duckdns-token/duckdns-token.env`.

```bash
kubectl apply --kustomize='secrets/duckdns-token/kustomization.yaml'
```
