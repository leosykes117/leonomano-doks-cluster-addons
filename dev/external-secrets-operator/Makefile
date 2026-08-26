SHELL := /bin/bash

NAMESPACE ?= external-secrets
KUSTOMIZE := kustomize
CRD       := clustersecretstores.external-secrets.io

.PHONY: render apply install-crds diff delete clean

## Render the Helm chart (delegates to base)
render:
	$(MAKE) -C ../../base/external-secrets-operator render

## ESO CRDs are not bundled by "helm template"; install separately.
install-crds:
	@echo "📦 Installing ESO CRDs..."
	kubectl apply -f "https://raw.githubusercontent.com/external-secrets/external-secrets/v0.20.2/deploy/crds/bundle.yaml" --server-side
	@echo "⏳ Waiting for CRD $(CRD) to be Established..."
	kubectl wait --for=condition=Established crd/$(CRD) --timeout=90s

# the "eso-dev-aws-creds" Secret referenced by
# resources/cluster-secret-store-aws-ssm.yaml is NOT created here
apply: render install-crds
	@echo "🚀 Applying ESO manifests (Helm + Kustomize)..."
	kubectl create namespace $(NAMESPACE) 2>/dev/null || true
	$(KUSTOMIZE) build . | kubectl apply -f -

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/external-secrets --timeout=180s || true

## Show diff against cluster
diff: render
	@echo "🔍 Diffing ESO manifests..."
	$(KUSTOMIZE) build . | kubectl diff -f - | tee diff.log || true

## Delete ESO resources
delete:
	@echo "🧨 Deleting ESO resources..."
	$(KUSTOMIZE) build . | kubectl delete -f - --ignore-not-found=true

## Clean generated files
clean:
	$(MAKE) -C ../../base/external-secrets-operator clean
	rm -f diff.log