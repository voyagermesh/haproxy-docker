SHELL=/bin/bash -o pipefail

REGISTRY   ?= ghcr.io/voyagermesh
BIN        ?= haproxy
IMAGE      := $(REGISTRY)/$(BIN)
GIT_TAG    ?= $(shell git describe --exact-match --abbrev=0 2>/dev/null || echo "")
TAG        := $(GIT_TAG)-alpine

DOCKER_PLATFORMS := linux/amd64 linux/arm64
PLATFORM         ?= $(firstword $(DOCKER_PLATFORMS))
VERSION          = $(TAG)_$(subst /,_,$(PLATFORM))

BASE_IMAGE       = haproxytech/haproxy-alpine:$(GIT_TAG)

BUILD_DIRS := bin

$(BUILD_DIRS):
	@mkdir -p $@

container-%:
	@$(MAKE) container \
	    --no-print-directory \
	    PLATFORM=$(subst _,/,$*)

push-%:
	@$(MAKE) push \
	    --no-print-directory \
	    PLATFORM=$(subst _,/,$*)

all-container: $(addprefix container-, $(subst /,_,$(DOCKER_PLATFORMS)))

all-push: $(addprefix push-, $(subst /,_,$(DOCKER_PLATFORMS)))

.PHONY: container
container: $(BUILD_DIRS)
	@echo "container: $(IMAGE):$(VERSION)"
	@rm -rf bin/.dockerfile;                             \
	sed                                                  \
	    -e 's|{BASE_IMAGE}|$(BASE_IMAGE)|g'        \
	    Dockerfile > bin/.dockerfile;                    \
	docker buildx build --platform $(PLATFORM) --load --pull -t $(IMAGE):$(VERSION) -f bin/.dockerfile .
	@echo

push: container
	@docker push $(IMAGE):$(VERSION)
	@echo "pushed: $(IMAGE):$(VERSION)"
	@echo

# https://stackoverflow.com/a/3732456
VER_MAJOR := $(shell echo $(GIT_TAG) | cut -f1 -d.)
VER_MINOR := $(shell echo $(GIT_TAG) | cut -f2 -d.)
Mm        := $(VER_MAJOR).$(VER_MINOR)-alpine

.PHONY: docker-manifest
docker-manifest:
	docker manifest create -a $(IMAGE):$(TAG) $(foreach PLATFORM,$(DOCKER_PLATFORMS),$(IMAGE):$(TAG)_$(subst /,_,$(PLATFORM)))
	docker manifest push $(IMAGE):$(TAG)
	docker manifest create -a $(IMAGE):$(Mm) $(foreach PLATFORM,$(DOCKER_PLATFORMS),$(IMAGE):$(TAG)_$(subst /,_,$(PLATFORM)))
	docker manifest push $(IMAGE):$(Mm)

.PHONY: release
release:
	@$(MAKE) all-push docker-manifest --no-print-directory

.PHONY: version
version:
	@echo version=$(VERSION)

.PHONY: fmt
fmt:
	@find . -path ./vendor -prune -o -name '*.sh' -exec shfmt -l -w -ci -i 4 {} \;

.PHONY: verify
verify: fmt
	@if !(git diff --exit-code HEAD); then \
		echo "files are out of date, run make fmt"; exit 1; \
	fi

.PHONY: ci
ci: verify

# make and load docker image to kind cluster
.PHONY: push-to-kind
push-to-kind: container
	@echo "Loading docker image into kind cluster...."
	@kind load docker-image $(IMAGE):$(VERSION)
	@echo "Image has been pushed successfully into kind cluster."
