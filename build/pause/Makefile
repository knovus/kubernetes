# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

.PHONY: all push container clean orphan all-push push-manifest

REGISTRY ?= knovus
IMAGE = $(REGISTRY)/pause
IMAGE_WITH_ARCH = $(IMAGE)

TAG = 3.3
REV = $(shell git describe --contains --always --match='v*')

# Architectures supported: amd64, arm, arm64, ppc64le and s390x
ARCH ?= amd64

ALL_ARCH = amd64 386

CFLAGS = -Os -Wall -Werror -static -DVERSION=v$(TAG)-$(REV)
KUBE_CROSS_IMAGE ?= knovus/kube-cross
KUBE_CROSS_VERSION ?= $(shell cat ../build-image/cross/VERSION)

BIN = pause
SRCS = pause.c

ifeq ($(ARCH),amd64)
	TRIPLE ?= x86_64-linux-gnu
endif

ifeq ($(ARCH),386)
	TRIPLE ?= i686-linux-gnu
endif

# If you want to build AND push all containers, see the 'all-push' rule.
all: all-container

all-push: all-push-images push-manifest

push-manifest:
	docker manifest create --amend $(IMAGE):$(TAG) $(shell echo $(ALL_ARCH) | sed -e "s~[^ ]*~$(IMAGE)\-&:$(TAG)~g")
	set -x; for arch in $(ALL_ARCH); do docker manifest annotate --arch $${arch} ${IMAGE}:${TAG} ${IMAGE}-$${arch}:${TAG}; done
	docker manifest push --purge ${IMAGE}:${TAG}

sub-container-%:
	$(MAKE) ARCH=$* container

sub-push-%:
	$(MAKE) ARCH=$* push

all-container: $(addprefix sub-container-,$(ALL_ARCH))

all-push-images: $(addprefix sub-push-,$(ALL_ARCH))

build: bin/$(BIN)-$(ARCH)

bin/$(BIN)-$(ARCH): $(SRCS)
	mkdir -p bin
	docker run --rm -u $$(id -u):$$(id -g) -v $$(pwd):/build \
		$(KUBE_CROSS_IMAGE):$(KUBE_CROSS_VERSION) \
		/bin/bash -c "\
			cd /build && \
			$(TRIPLE)-gcc $(CFLAGS) -o $@ $^ && \
			$(TRIPLE)-strip $@"

container: .container-$(ARCH)
.container-$(ARCH): bin/$(BIN)-$(ARCH)
	DOCKER_CLI_EXPERIMENTAL=enabled docker buildx build --pull --platform linux/$(ARCH) -t $(IMAGE_WITH_ARCH):$(TAG) --build-arg ARCH=$(ARCH) --push .
	touch $@

push: .push-$(ARCH)
.push-$(ARCH): .container-$(ARCH)
	docker push $(IMAGE_WITH_ARCH):$(TAG)
	touch $@

# Useful for testing, not automatically included in container image
orphan: bin/orphan-$(ARCH)
bin/orphan-$(ARCH): orphan.c
	mkdir -p bin
	docker run -u $$(id -u):$$(id -g) -v $$(pwd):/build \
		$(KUBE_CROSS_IMAGE):$(KUBE_CROSS_VERSION) \
		/bin/bash -c "\
			cd /build && \
			$(TRIPLE)-gcc $(CFLAGS) -o $@ $^ && \
			$(TRIPLE)-strip $@"

clean:
	rm -rf .container-* .push-* bin/
