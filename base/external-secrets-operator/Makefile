SHELL := /bin/bash

NAMESPACE := external-secrets
RELEASE   := external-secrets
CHART     := external-secrets/external-secrets
CHART_VERSION ?= 0.20.2

VALUES    := helm/values.yaml
RENDERED  := rendered/external-secrets.yaml

.PHONY: render clean

## Render Helm chart to plain YAML (consumed by dev/external-secrets-operator via kustomize base)
render:
	@echo "🔧 Rendering ESO Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	@helm repo add external-secrets https://charts.external-secrets.io >/dev/null 2>&1 || true
	@helm repo update external-secrets >/dev/null 2>&1 || true
	helm template $(RELEASE) $(CHART) \
		--version $(CHART_VERSION) \
		-n $(NAMESPACE) \
		-f $(VALUES) \
		| tee $(RENDERED)
	@echo "Rendered: $(RENDERED)"

## Clean generated files
clean:
	@echo "🧹 Cleaning rendered files..."
	rm -f $(RENDERED)
