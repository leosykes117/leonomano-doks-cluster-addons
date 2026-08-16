SHELL := /bin/bash

NAMESPACE := cert-manager
RELEASE   := cert-manager
CHART     := jetstack/cert-manager
CHART_VERSION ?= v1.19.1

VALUES    := helm/values.yaml
RENDERED  := rendered/cert-manager.yaml

.PHONY: render clean

## Render Helm chart to plain YAML (consumed by dev/cert-manager via kustomize base)
render:
	@echo "🔧 Rendering cert-manager Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	@helm repo add jetstack https://charts.jetstack.io >/dev/null 2>&1 || true
	@helm repo update jetstack >/dev/null 2>&1 || true
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
