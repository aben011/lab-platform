# lab-platform

Platform-level infra and cluster tooling.

## Bootstrapping ArgoCD

```bash
kubectl apply --kustomize='argocd-controller'
```

## Secret: lab-gateway-tls (self-signed, temporary)

The `websecure` listener on `lab-gateway` references a Secret,
`lab-gateway-tls`, that is **not created by anything in this repo**.
Gateway API requires a real Secret object for a `Terminate`-mode HTTPS
listener — there's no "just enable TLS" flag, so until cert-manager +
the DuckDNS ACME issuer are wired back in (paused for now), this is a
manually generated, self-signed, unverified certificate. Browsers will
show a trust warning — expected, ignore it or trust the cert locally.

Not committed to git: it's a generated artifact, not configuration, and
regenerating it is cheap and expected once real certs replace it.

### Generate and apply, once, manually

```bash
openssl req -x509 -nodes -days 365 \
  -newkey ec \
  -pkeyopt ec_paramgen_curve:prime256v1 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=*.ab3704-k.duckdns.org" \
  -addext "subjectAltName=DNS:*.ab3704-k.duckdns.org"

kubectl create secret tls lab-gateway-tls \
  --namespace kube-system \
  --cert=tls.crt --key=tls.key

rm tls.crt tls.key
```

### Replacing this later with a real cert-manager-issued cert

Once the DuckDNS webhook + ClusterIssuer are back in place, swap this
listener's `certificateRefs` to point at a Secret name managed by a
`Certificate` resource instead (cert-manager owns creating/renewing that
Secret automatically). Delete this manually-created Secret at that
point so there's no ambiguity about which one is authoritative.
