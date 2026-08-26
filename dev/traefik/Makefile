SHELL := /bin/bash

NAMESPACE ?= traefik
KUSTOMIZE := kustomize

.PHONY: render apply diff delete clean

## Render the Helm chart (delegates to base)
render:
	$(MAKE) -C ../../base/traefik render

install-crds:
	@echo "📦 Installing Traefik CRDs..."
	helm show crds traefik/traefik | kubectl apply -f - --server-side
	@echo "⏳ Waiting for CRDs to be Established..."
	kubectl wait --for=condition=Established crd/ingressroutes.traefik.io --timeout=90s
	kubectl wait --for=condition=Established crd/middlewares.traefik.io --timeout=90s

## Apply patched manifests with Kustomize
# Depends on the cert-manager addon already being applied
# (ClusterIssuer "letsencrypt-staging" must already exist).
apply: render install-crds
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
