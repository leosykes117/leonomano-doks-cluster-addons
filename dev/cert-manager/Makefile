SHELL := /bin/bash

NAMESPACE ?= cert-manager
KUSTOMIZE := kustomize
CRD       := clusterissuers.cert-manager.io

.PHONY: render apply diff delete clean

## Render the Helm chart (delegates to base)
render:
	$(MAKE) -C ../../base/cert-manager render

## Apply patched manifests with Kustomize
# Depends on the external-secrets-operator addon already being applied
# (ClusterSecretStore "aws-dev-parameter-store" must already exist).
apply: render
	@echo "🚀 Applying cert-manager manifests (Helm + Kustomize)..."
	kubectl create namespace $(NAMESPACE) 2>/dev/null || true

	@echo "1️⃣  Applying operator (CRDs + Deployments + webhook)..."
	$(KUSTOMIZE) build ../../base/cert-manager | kubectl apply -f -

	@echo "⏳ Waiting for CRD $(CRD) to be Established..."
	kubectl wait --for=condition=Established crd/$(CRD) --timeout=90s

	@echo "⏳ Waiting for the cert-manager webhook to be ready..."
	kubectl -n $(NAMESPACE) rollout status deploy/cert-manager-webhook --timeout=120s

	@echo "2️⃣  Applying full overlay (retrying: ESO's webhook may need a moment to be network-reachable)..."
	@n=0; until [ $$n -ge 6 ]; do \
		$(KUSTOMIZE) build . | kubectl apply -f - && break; \
		n=$$((n+1)); \
		echo "   ⚠️  Attempt $$n/6 failed, retrying in 5s..."; \
		sleep 5; \
	done; \
	if [ $$n -ge 6 ]; then echo "❌ apply failed after 6 attempts"; exit 1; fi

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/cert-manager --timeout=180s || true

## Show diff against cluster
diff: render
	@echo "🔍 Diffing cert-manager manifests..."
	$(KUSTOMIZE) build . | kubectl diff -f - | tee diff.log || true

## Delete cert-manager resources
delete:
	@echo "🧨 Deleting cert-manager resources..."
	$(KUSTOMIZE) build . | kubectl delete -f - --ignore-not-found=true

## Clean generated files
clean:
	$(MAKE) -C ../../base/cert-manager clean
	rm -f diff.log