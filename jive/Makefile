#
# Makefile for building and publishing the `tango_jive` image.
#
# Targets:
# 	build: Build the image
# 	publish: Publish the image to Dockerhub / cloud.docker.com
#	clean: Remove the image from the local machine
#	jive: Example target that can be used to give an Jive gui connected
#         to a Tango database container.
#

CRED=\033[0;31m
CBLUE=\033[0;34m
CEND=\033[0m
LINE=$(shell printf '=%.0s' {1..70})

# Set default docker registry user.
ifeq ($(strip $(DOCKER_REGISTRY_USER)),)
  DOCKER_REGISTRY_USER = bmort
endif

# Get image name and version from the release file
NAME=$(shell awk -F= '/^name=/{print $$2}' release)
VERSION=$(shell awk -F= '/^release=/{print $$2}' release)
IMAGE=$(DOCKER_REGISTRY_USER)/$(NAME)

IMAGES := $(shell docker images --format '{{.Repository}}:{{.Tag}}' | grep $(IMAGE))
TANGO_DB_ID=$(shell docker ps -f name=tango_database -f status=running -q)
IP=$(shell ifconfig en7 | grep inet | awk '$$1=="inet" {print $$2}')

all: build

build: build_version tag_latest

help:
	@echo "$(IMAGE)"
	@echo "$(VERSION)"

build_version:
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Building Docker image:$(CEND) $(IMAGE):$(VERSION)"
	@echo "$(CRED)$(LINE)$(CEND)"
	docker build -t ${IMAGE}:${VERSION} .

tag_latest:
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Tagging image:$(CEND) $(IMAGE):latest"
	@echo "$(CRED)$(LINE)$(CEND)"
	docker tag $(IMAGE):$(VERSION) $(IMAGE):latest

publish: build tag_latest
	@echo "$(CRED)* $(CBLUE)Publishing image(s) to Dockerhub ...$(CEND)"
	docker push $(IMAGE):$(VERSION)
	docker push $(IMAGE):latest

clean:
ifeq ($(strip $(IMAGES)),)
	@echo "$(CRED)No images need removing!$(CEND)"
else
	docker image rm $(IMAGES)
endif

prompt:
ifeq ($(strip $(IMAGES)),)
	@echo "$(CRED)ERROR: Image '$(IMAGE):latest' not found!$(CEND)"
else
ifeq ($(strip $(TANGO_DB_ID)),)
	@echo "$(CRED)ERROR: Unable to start prompt, tango_database containernot found!$(CEND)"
else
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Starting interactive prompt (type 'exit' to quit)$(CEND)"
	@echo "$(CRED)$(LINE)$(CEND)"
	@docker run --rm -ti \
		--name tango_dev_prompt \
		--network=container:$(TANGO_DB_ID) \
		-e TANGO_HOST=localhost:10000 \
		--entrypoint=/bin/bash \
		$(IMAGE):latest
endif
endif


socat: stop_socat
	@socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$$DISPLAY\" &

# interface might not always be en7 .. (en0 ?)

jive: socat
ifeq ($(strip $(IMAGES)),)
	@echo "$(CRED)ERROR: Image '$(IMAGE):latest' not found!$(CEND)"
else
ifeq ($(strip $(TANGO_DB_ID)),)
	@echo "$(CRED)ERROR: Unable to start prompt, tango_database containernot found!$(CEND)"
else
	@echo "$(TANGO_DB_ID)"
	@docker run --rm -ti \
		-e DISPLAY=$(IP):0 \
		-e TANGO_HOST=localhost:10000 \
		--network=container:$(TANGO_DB_ID) \
		--name tango_jive \
		--entrypoint=/bin/bash \
		$(IMAGE):latest
endif
endif

stop_socat:
	@killall socat

test_gui: socat
ifeq ($(strip $(IP)),)
	@echo "$(CRED)ERROR: IP addess for display not known!$(CEND)"
else
	# xhost+$(IP)
	@docker run --rm -ti \
		-e DISPLAY=$(IP):0 \
		--name test_gui \
		gns3/xeyes
endif
