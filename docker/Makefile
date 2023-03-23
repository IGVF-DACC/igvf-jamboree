REGISTRY?=leepc12
IMAGE_NAME?=test-jupyterhub
VERSION=v0.2.0

PARENT_DOCKER=jupyter/base-notebook

all: build

.PHONY: build
build:
	DOCKER_BUILDKIT=1 docker build \
	--tag ${IMAGE_NAME}:${VERSION} \
	--tag ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
	--build-arg PARENT_DOCKER=${PARENT_DOCKER} \
	--build-arg BUILDKIT_INLINE_CACHE=1 \
	--cache-from ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
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
