SHELL := /bin/bash

NAMESPACE ?= external-dns
KUSTOMIZE := kustomize
# CRD       := clusterissuers.cert-manager.io

.PHONY: render apply diff delete clean

## Render the Helm chart (delegates to base)
render:
	$(MAKE) -C ../../base/external-dns render

## Apply patched manifests with Kustomize
# Depends on the external-secrets-operator addon already being applied
# (ClusterSecretStore "aws-dev-parameter-store" must already exist).
apply: render
	@echo "🚀 Applying external-dns manifests (Helm + Kustomize)..."
	kubectl create namespace $(NAMESPACE) 2>/dev/null || true

	@echo "1️⃣ Applying full overlay (ExternalSecret + operator)..."
	$(KUSTOMIZE) build . | kubectl apply -f -

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/external-dns --timeout=180s || true

## Show diff against cluster
diff: render
	@echo "🔍 Diffing external-dns manifests..."
	$(KUSTOMIZE) build . | kubectl diff -f - | tee diff.log || true

## Delete external-dns resources
delete:
	@echo "🧨 Deleting external-dns resources..."
	$(KUSTOMIZE) build . | kubectl delete -f - --ignore-not-found=true

## Clean generated files
clean:
	$(MAKE) -C ../../base/external-dns clean
	rm -f diff.log