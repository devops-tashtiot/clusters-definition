# CLAUDE.md — clusters-definition

This repo holds environment-specific Helm values overrides for shared cluster infrastructure (ingress, secrets, tunnel). It is the configuration layer for the `clusters-provision`/`clusters-definition` pair — the counterpart to `devtools-provision`/`devtools-definition`, split out because devtools can depend on this infra being ready first (e.g. bitbucket's `ExternalSecret` needs `external-secrets-operator` running).

## Role in the Architecture

ArgoCD's ApplicationSet (defined in this repo's `applicationset.yaml`, bootstrapped by `application.yaml`) deploys each cluster-infra tool using **two sources**:

1. The Helm chart from `clusters-provision` (base chart + default `values.yaml`)
2. A reference named `$definition` pointing to this repo

Values are merged: `clusters-provision/clusters/<tool>/values.yaml` is applied first, then `clusters-definition/clusters/<tool>/values.yaml` overrides on top.

This means: **don't duplicate base values here — only override what differs per environment.**

The `devtools-labs` `minikube` module registers this repo's `clusters-applicationset` *before* the `devtools-applicationset`, and blocks until `ingress-nginx`, `cloudflared`, `external-secrets-operator`, and `rhbk` report Synced+Healthy — see that repo's `CLAUDE.md`. `rhbk` is in that wait-list (unlike most cluster-infra tools) because the devtools ApplicationSet includes ArgoCD's own OIDC values, which assume `rhbk`'s realm/client already exist.

## Repository Structure

```
.
├── application.yaml       # ArgoCD Application — bootstraps the ApplicationSet from GitHub
├── applicationset.yaml    # ApplicationSet — auto-discovers clusters/* in clusters-provision
└── clusters/
    └── <tool>/
        └── values.yaml   # Environment-specific overrides for the tool's Helm chart
```

Currently tracked tools: `cloudflared`, `ingress-nginx`, `external-secrets-operator`, `rhbk`.

## Adding Overrides for a New Cluster-Infra Tool

1. Create `clusters/<toolname>/values.yaml`
2. Include only values that differ from the defaults in `clusters-provision/clusters/<toolname>/values.yaml`
3. Commit and push to `main` — ArgoCD will pick up the change automatically on the next sync

## What Belongs Here vs. clusters-provision

| Concern | Where it lives |
|---|---|
| Chart dependency, chart version | `clusters-provision/clusters/<tool>/Chart.yaml` |
| Default values | `clusters-provision/clusters/<tool>/values.yaml` |
| Environment-specific overrides | `clusters-definition/clusters/<tool>/values.yaml` (this repo) |
| Cluster infrastructure / bootstrap ordering | `devtools-labs` |
