# clusters-definition

Environment-specific Helm values overrides for the shared cluster infrastructure in
[`clusters-provision`](https://github.com/devops-tashtiot/clusters-provision) — the
configuration half of that provision/definition pair. The devtools equivalent is
[`devtools-provision`](https://github.com/devops-tashtiot/devtools-provision)/
[`devtools-definition`](https://github.com/devops-tashtiot/devtools-definition).

## Why a separate repo instead of one

Devtools can depend on this infrastructure being ready first — e.g. Bitbucket's
`ExternalSecret` needs `external-secrets-operator` running before it can start. Splitting
infra config out from devtool config lets `devtools-labs` register and wait on this
ApplicationSet before the devtools one, without conflating "what infra exists" with "what
devtools exist."

## How it works

ArgoCD's `ApplicationSet` (`applicationset.yaml`) auto-discovers every tool directory under
`clusters-provision/clusters/*` and merges two Helm value sources per tool:

1. `clusters-provision/clusters/<tool>/values.yaml` — defaults, same across every
   environment
2. `clusters/<tool>/values.yaml` (this repo) — environment-specific overrides, applied on
   top

Only override what differs per environment here — don't duplicate base values.

## Currently tracked tools

`cloudflared`, `ingress-nginx`, `external-secrets-operator`, `rhbk` — matching
`clusters-provision/clusters/*` exactly (ArgoCD requires the identical directory name on
both sides).

See [`CLAUDE.md`](./CLAUDE.md) for the full architecture writeup, and the
`add-cluster-provision` skill for onboarding a new tool.
