SHELL := /bin/bash

NAMESPACE ?= traefik
KUSTOMIZE := kustomize

.PHONY: render apply diff delete clean

## Render the Helm chart (delegates to base)
render:
	$(MAKE) -C ../../base/traefik render

## Apply patched manifests with Kustomize
# Depends on the cert-manager addon already being applied
# (ClusterIssuer "letsencrypt-staging" must already exist).
apply: render
	@echo "🚀 Applying Traefik manifests (Helm + Kustomize)..."
	kubectl create namespace $(NAMESPACE) 2>/dev/null || true
	$(KUSTOMIZE) build . | kubectl apply -f -

install: apply
	kubectl -n $(NAMESPACE) rollout status deploy/traefik --timeout=180s || true

## Show diff against cluster
diff: render
	@echo "🔍 Diffing Traefik manifests..."
	$(KUSTOMIZE) build . | kubectl diff -f - | tee diff.log || true

## Delete Traefik resources
delete:
	@echo "🧨 Deleting Traefik resources..."
	$(KUSTOMIZE) build . | kubectl delete -f - --ignore-not-found=true

## Clean generated files
clean:
	$(MAKE) -C ../../base/traefik clean
	rm -f diff.log
