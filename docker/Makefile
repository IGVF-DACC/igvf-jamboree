REGISTRY?=leepc12
IMAGE_NAME?=igvf-jamboree-2024
CACHE_VERSION=v1.0.0
VERSION=v1.0.1

PARENT_DOCKER=jupyter/base-notebook@sha256:0ec801994602a53d3917d0f07ccf4474a1ddf5ae419483b24681a3c42abc4ce5

all: build

.PHONY: build
build:
	DOCKER_BUILDKIT=1 docker build \
	--tag ${IMAGE_NAME}:${VERSION} \
	--tag ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
	--build-arg PARENT_DOCKER=${PARENT_DOCKER} \
	--build-arg BUILDKIT_INLINE_CACHE=1 \
	--cache-from ${REGISTRY}/${IMAGE_NAME}:${CACHE_VERSION} \
	.

.PHONY: build-no-cache
build-no-cache:
	DOCKER_BUILDKIT=1 docker build \
	--tag ${IMAGE_NAME}:${VERSION} \
	--tag ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
	--build-arg PARENT_DOCKER=${PARENT_DOCKER} \
	.

.PHONY: test
test:
	docker run --rm -it -p 8888:8888 ${REGISTRY}/${IMAGE_NAME}:${VERSION} bash

.PHONY: test_bash
test_bash:
	docker run --rm -it -p 8888:8888 ${REGISTRY}/${IMAGE_NAME}:${VERSION} bash

.PHONY: push
push:
	docker push ${REGISTRY}/${IMAGE_NAME}:${VERSION}
