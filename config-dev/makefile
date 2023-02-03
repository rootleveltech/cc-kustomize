#! /usr/bin/make -f

-include makefile.config
-include makefile.config.local

SHELL := /usr/local/bin/bash -O globstar

.PHONY: clean default debug_kustomize debug_personalize_kustomizations kustomized_manifests purge test_yaml venv

default: test_yaml


KUSTOMIZATIONS = $(shell ls src/**/kustomization.yaml)

KUSTOMIZED_MANIFESTS = $(patsubst src/%/kustomization.yaml, %/kustomized-manifest.yaml, $(KUSTOMIZATIONS))

kustomized_manifests: $(KUSTOMIZED_MANIFESTS)

%/kustomized-manifest.yaml: src/%/kustomization.yaml
	@echo -n "Creating $@ with container: "
	@$(eval RAND := $(shell /bin/bash -c "echo $$RANDOM"))
	@mkdir -p "$(@D)"
	@docker create \
		--name=$(kustomize_name)-$(RAND) \
        --rm=true \
        --tty=true \
        --volume="$(shell pwd -P)/kustomize-helm:/kustomize-helm:ro,Z" \
        $(kustomize_namespace)/$(kustomize_image):$(kustomize_tag) \
        	/kustomize-helm/kustomizer "/home/argocd/src/$(@D)"
	@docker cp src $(kustomize_name)-$(RAND):/home/argocd/src
	@printf '%s\n%s\n' '# This is an auto-generated file. DO NOT EDIT' '---' > "$@"
	@docker start --attach=true $(kustomize_name)-$(RAND) | sed 's/\r$$//' >> "$@"

debug_kustomize:
	docker create \
		--env GIT_TRACE=true \
		--env GIT_CURL_VERBOSE=true \
		--env GIT_SSH_COMMAND="ssh -vvv" \
		--env GIT_TRACE_PACK_ACCESS=true \
		--env GIT_TRACE_PACKFILE=true \
		--env GIT_TRACE_PACKET=true \
		--env GIT_TRACE_PERFORMANCE=true \
		--env GIT_TRACE_SETUP=true \
		--env GIT_TRACE_SHALLOW=true \
		--env HELM_GIT_DEBUG=1 \
		--env HELM_PLUGINS=/home/argocd/.local/share/helm/plugins \
		--name=$(kustomize_name) \
		--rm=true \
		--tty=true \
		--volume="$(shell pwd)/kustomize-helm:/kustomize-helm:ro,Z" \
		argoproj/argocd /bin/bash
	docker cp src $(kustomize_name):/home/argocd/src
	docker start \
		--attach=true \
		--interactive=true \
		$(kustomize_name)

debug_personalize_kustomizations: update-resource-urls.sh
	@for k in $(KUSTOMIZATIONS); do \
		./update-resource-urls.sh "$(kustomize_scm_prefix)" "$(kustomize_scm_ref)" "$$k"; \
	done

test_yaml: .venv
	.venv/bin/python -m yamllint .

.venv:
	python3 -m venv .venv
	.venv/bin/python -m pip install --upgrade pip
	.venv/bin/python -m pip install yamllint

venv: .venv

clean:
	@rm -rf .venv

purge: clean
	find . -name 'kustomized-manifest.yaml' -delete
