# leonomano-doks-cluster-addons

Kubernetes-native cluster addon configuration for the DigitalOcean Kubernetes
(DOKS) cluster: cert-manager, Traefik, and External Secrets Operator (ESO).

## Why this repo exists

Cloud-provider resources (the DOKS cluster itself, IAM users/keys, SSM
parameters) are managed by Terraform/Terragrunt in
[leonomano-doks-tf-modules](https://github.com/leosykes117/leonomano-doks-tf-modules)
and [leonomano-doks-tg-infra](https://github.com/leosykes117/leonomano-doks-tg-infra).

Everything that is a Kubernetes-native resource (Helm releases, CRs like
`ClusterIssuer`/`ExternalSecret`/`Certificate`, namespaces) lives here instead,
managed with Helm + Kustomize + kubectl. This separation follows the
GitOps best practice of keeping infrastructure-as-code and cluster-manifest
config in different repositories (see
[Argo CD's Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
and [Flux's repository-structure guide](https://fluxcd.io/flux/guides/repository-structure/)),
and sets this repo up to be consumed by Argo CD directly later on, without
restructuring.

## Layout

```
base/<addon>/         # Helm chart render + chart-level values, environment-agnostic
  Makefile              make render -> helm template chart into rendered/
  helm/values.yaml
  kustomization.yaml    references rendered/<chart>.yaml

dev/<addon>/          # Per-environment overlay
  Makefile              make apply/diff/delete, composes base + env resources
  kustomization.yaml    bases: [../../base/<addon>], resources: [resources/*.yaml]
  resources/*.yaml      environment-specific Custom Resources (ClusterIssuer, etc.)
```

Addons are applied in dependency order (each depends on the previous one's
CRDs/resources already existing on the cluster):

```
external-secrets-operator -> cert-manager -> traefik
```

## Important: no secrets are stored in this repo

The `eso-<env>-aws-creds` Secret referenced by
`dev/external-secrets-operator/resources/cluster-secret-store-aws-ssm.yaml` is
**not** defined here. Its values (an AWS IAM access key created by the
`external-secrets-operator` Terraform module) are read from Terraform outputs
and materialized with `kubectl create secret` by the CI pipeline at apply
time. See `leonomano-doks-tg-infra`'s `provision-doks.yml` workflow.

## Usage

```bash
cd dev/external-secrets-operator && make apply
cd dev/cert-manager && make apply
cd dev/traefik && make apply
```
