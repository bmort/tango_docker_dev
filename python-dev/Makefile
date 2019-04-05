#
# Makefile for building and publishing the `tango_python_dev` image.
#
# Targets:
# 	build: Build the image. Creates two tags, :<current version> and :<latest>
# 	publish: Build and publish the image to Dockerhub / cloud.docker.com
#	clean: Remove the image from the local machine
#	prompt: Example target giving a bash prompt running in the container
#           and connected to a tango database deployment.
# 	itango: Example target giving an iTango prompt running in the container
#           and connected to a tango database deployment.
#

CRED=\033[0;31m
CBLUE=\033[0;34m
CEND=\033[0m
LINE:=$(shell printf '=%.0s' {1..70})

# Set default docker registry user.
ifeq ($(strip $(DOCKER_REGISTRY_USER)),)
  DOCKER_REGISTRY_USER = bmort
endif

# Get image name and version from the release file
NAME=$(shell awk -F= '/^name=/{print $$2}' release)
VERSION=$(shell awk -F= '/^release=/{print $$2}' release)
IMAGE=$(DOCKER_REGISTRY_USER)/$(NAME)

IMAGES := $(shell docker images --format '{{.Repository}}:{{.Tag}}' | grep $(IMAGE))
TANGO_DB_ID:=$(shell docker ps -f name=tango_database -f status=running -q)
IP:=$(shell ifconfig en7 | grep inet | awk '$$1=="inet" {print $$2}')
DEV_CONTAINER:=$(shell docker ps -f name=tango_dev_prompt -q)

ifeq ($(stip $(IMAGES)),"")
	IMAGE_EXISTS = 0
else
	IMAGE_EXISTS = 1
endif
ifeq ($(stip $(TANGO_DB_ID)),"")
	TANGO_DB_EXISTS = 0
else
	TANGO_DB_EXISTS = 1
endif
ifeq ($(strip $(DEV_CONTAINER)),"")
	DEV_CONTAINER_STARTED = 0
else
	DEV_CONTAINER_STARTED = 1
endif


.PHONY: all
all: build

.PHONY: build
build: build_version tag_latest

.PHONY: build_version
build_version:
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Building Docker image:$(CEND) $(IMAGE):$(VERSION)"
	@echo "$(CRED)$(LINE)$(CEND)"
	docker build -t ${IMAGE}:${VERSION} .

.PHONY: tag_latest
tag_latest:
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Tagging image:$(CEND) $(IMAGE):latest"
	@echo "$(CRED)$(LINE)$(CEND)"
	docker tag $(IMAGE):$(VERSION) $(IMAGE):latest

.PHONY: publish
publish: build tag_latest
	@echo "$(CRED)* $(CBLUE)Publishing image(s) to Dockerhub ...$(CEND)"
	docker push $(IMAGE):$(VERSION)
	docker push $(IMAGE):latest

.PHONY: clean
clean:
ifeq ($(strip $(IMAGES)),)
	@echo "$(CRED)No images need removing!$(CEND)"
else
	docker image rm $(IMAGES)
endif

.PHONY: prompt
prompt:
ifeq ($(IMAGE_EXISTS),0)
	@echo "$(CRED)ERROR: Image '$(IMAGE):latest' not found!$(CEND)"
endif
ifeq ($(TANGO_DB_EXISTS),0)
	@echo "$(CRED)ERROR: Unable to start prompt, tango_database containernot found!$(CEND)"
endif
ifeq ($(DEV_CONTAINER_STARTED),1)
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Connecting dev container to start iTango"\
	 	  "(type 'exit' to quit)$(CEND)"
	@echo "$(CRED)$(LINE)$(CEND)"
	@docker exec -it $(DEV_CONTAINER) /bin/bash
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

.PHONY: itango
itango:
ifeq ($(IMAGE_EXISTS),0)
	@echo "$(CRED)ERROR: Image '$(IMAGE):latest' not found!$(CEND)"
endif
ifeq ($(TANGO_DB_EXISTS),0)
	@echo "$(CRED)ERROR: Unable to start prompt, tango_database containernot found!$(CEND)"
endif
ifeq ($(DEV_CONTAINER_STARTED),1)
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Connecting dev container to start iTango (type 'exit' to quit)$(CEND)"
	@echo "$(CRED)$(LINE)$(CEND)"
	@docker exec -it $(DEV_CONTAINER) itango3
else
	@echo "$(CRED)$(LINE)$(CEND)"
	@echo "$(CBLUE)Starting iTango prompt (type 'exit' to quit)$(CEND)"
	@echo "$(CRED)$(LINE)$(CEND)"
	@docker run --rm -ti \
		--name tango_dev_prompt \
		--network=container:$(TANGO_DB_ID) \
		-e TANGO_HOST=localhost:10000 \
		--entrypoint=/usr/local/bin/pipenv \
		$(IMAGE):latest \
		run itango3
endif
