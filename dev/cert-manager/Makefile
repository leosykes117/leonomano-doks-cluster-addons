SHELL := /bin/bash

NAMESPACE ?= cert-manager
KUSTOMIZE := kustomize

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
	$(KUSTOMIZE) build . | kubectl apply -f -

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
