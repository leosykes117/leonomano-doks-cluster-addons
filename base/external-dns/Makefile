SHELL := /bin/bash

NAMESPACE := external-dns
RELEASE   := external-dns
CHART     := external-dns/external-dns
CHART_VERSION ?= 1.21.1

VALUES    := helm/values.yaml
RENDERED  := rendered/external-dns.yaml

.PHONY: render clean

## Render Helm chart to plain YAML (consumed by $env/external-dns via kustomize base)
render:
	@echo "🔧 Rendering external-dns Helm chart..."
	@mkdir -p $$(dirname "$(RENDERED)")
	@helm repo add $(RELEASE) https://kubernetes-sigs.github.io/external-dns/ || true
	@helm repo update $(RELEASE) >/dev/null 2>&1 || true
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
