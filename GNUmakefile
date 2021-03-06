# Official and semi-official architectures: https://github.com/docker-library/official-images#architectures-other-than-amd64
# Webproc for linux is available for amd64, i386 and arm
ARCHITECTURES=amd64 armv7 i386
IMAGE_NAME=outlyernet/dnsmasq-multiarch
# Dockerfile.in: Dockerfile template
DOCKERFILE_IN=Dockerfile.in

DOCKERFILES=$(addsuffix .Dockerfile,$(ARCHITECTURES))
# The colon confuses make, leave it for later
IMAGES_TARGET=$(addprefix $(IMAGE_NAME).latest-,$(ARCHITECTURES))
IMAGES=$(addprefix $(IMAGE_NAME):latest-,$(ARCHITECTURES))

# Download URLs take the form:
# https://github.com/jpillora/webproc/releases/download/$RELEASE/webproc_linux_$ARCH.gz
# e.g.
# https://github.com/jpillora/webproc/releases/download/0.2.2/webproc_linux_amd64.gz

# Translate the architecture names (resolution delayed to the actual rules)
# Docker Hub prefix:
# amd64 => amd64
# armv7 => arm32v7 (XXX: Or is it arm32v5?)
# i386 => i386
#DOCKER_PREFIX=$(shell echo $* | sed -e 's/armv7/arm32v7/' -e 's/armle/arm32v5/')
DOCKER_PREFIX=$(subst armv7,arm32v7,$*)
# Webproc's architecture name:
# amd64 => amd64
# armv7 => arm
# i386 => 386
WEBPROC_ARCH=$(subst armv7,arm,$(subst i386,386,$*))
# Alpine architecture name:
# amd64 => x86_64
# armv7 => armv7
# i386 => x86
ALPINE_ARCH=$(subst i386,x86,$(subst amd64,x86_64,$*))

all: $(DOCKERFILES) $(IMAGES_TARGET)

%.Dockerfile: $(DOCKERFILE_IN)
	sed -e 's#DOCKER_PREFIX=.*$$#DOCKER_PREFIX=$(DOCKER_PREFIX)#' \
		-e 's!ARCHITECTURE=.*$$!ARCHITECTURE=$(WEBPROC_ARCH)!' \
		-e 's!ALPINE_ARCH=.*$$!ALPINE_ARCH=$(ALPINE_ARCH)!' \
		-e 's/amd64 architecture\./$* architecture\./' $< > $@

$(IMAGE_NAME).latest-%: %.Dockerfile
	docker build -t $(subst .,:,$@) -f $< .

# Repository-specific stuff. Can only be used as-is by me

push-images:
	for image in $(IMAGES); do \
		docker push $$image ; \
	done

# The manifest doesn't include the EXTRA images
manifest: push-images
	env DOCKER_CLI_EXPERIMENTAL=enabled \
		docker manifest create $(IMAGE_NAME):latest \
			$(IMAGES)

# Forceful, but gets rid of trouble
# <https://github.com/docker/cli/issues/954>
push-manifest: manifest
	env DOCKER_CLI_EXPERIMENTAL=enabled \
		docker manifest push --purge $(IMAGE_NAME):latest

push: push-manifest

distclean:
	$(RM) $(DOCKERFILES)

.PHONY: all distclean