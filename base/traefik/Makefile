SHELL := /bin/bash

NAMESPACE := traefik
RELEASE   := traefik
CHART     := traefik/traefik
CHART_VERSION ?= 37.1.1

VALUES    := helm/values.yaml
RENDERED  := rendered/traefik.yaml

.PHONY: render clean

## Render Helm chart to plain YAML (consumed by dev/traefik via kustomize base)
render:
	@echo "🔧 Rendering Traefik Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	@helm repo add traefik https://traefik.github.io/charts >/dev/null 2>&1 || true
	@helm repo update traefik >/dev/null 2>&1 || true
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
