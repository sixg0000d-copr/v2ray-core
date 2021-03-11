MAKEFILE_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
SCM_ROOT := $(abspath $(MAKEFILE_DIR)/..)
# SPECS
SOURCE_URL := https://github.com/v2fly/v2ray-core
NAME := v2ray-core
TAG := $(shell cat $(MAKEFILE_DIR)/TAG)
VERSION := $(patsubst v%,%,$(TAG))
RELEASE := $(shell cat $(MAKEFILE_DIR)/RELEASE)
DATE := $(shell LC_ALL=en_US.UTF-8 date +'%a %b %d %Y')
# ENVIRONMENTS
spec := $(or $(spec), $(SCM_ROOT)/$(NAME).spec)
outdir := $(or $(outdir), $(SCM_ROOT)/outdir)
# PACKAGE MANAGER INSTALL COMMAND
ifeq ($(shell id -u),0)
pm_install := dnf -y install
else
pm_install := sudo dnf -y install
endif


all: srpm

init:
	$(pm_install) "@RPM Development Tools" "tar" "git" "sed" "golang >= 1.11"

$(outdir):
	[ -d "$(outdir)" ] || mkdir -p $(outdir)

spec: $(outdir) $(spec).in
	sed -e "s|@SOURCE_URL@|$(SOURCE_URL)|g" \
	    -e "s|@NAME@|$(NAME)|g" \
	    -e "s|@TAG@|$(TAG)|g" \
	    -e "s|@VERSION@|$(VERSION)|g" \
	    -e "s|@RELEASE@|$(RELEASE)|g" \
	    -e "s|@DATE@|$(DATE)|g" \
	    $(spec).in > $(outdir)/$(NAME).spec

sources: $(outdir) init
	# download source archive and vendor
	cd $(outdir) && \
	curl -L $(SOURCE_URL)/archive/$(TAG)/$(NAME)-$(VERSION).tar.gz -o $(NAME)-$(VERSION).tar.gz && \
	tar -xzf $(NAME)-$(VERSION).tar.gz && \
	cd $(NAME)-$(VERSION) && \
	go mod vendor
	# compress vendor
	cd $(outdir)/$(NAME)-$(VERSION) && \
	tar -czf "$(outdir)/vendor.tar.gz" "vendor"
	# build config and systemd unit files
	cp "$(outdir)/$(NAME)-$(VERSION)/release/config/config.json" "$(outdir)/config.json"
	sed -e "s|User=.*|DynamicUser=true|" "$(outdir)/$(NAME)-$(VERSION)/release/debian/v2ray.service" > "$(outdir)/v2ray.service"
	sed -e "s|User=.*|DynamicUser=true|" "$(outdir)/$(NAME)-$(VERSION)/release/debian/v2ray@.service" > "$(outdir)/v2ray@.service"
	sed -e "s|DynamicUser=true|User=v2ray\nDynamicUser=true|; s|-config|-confdir|g; s|/etc/v2ray/config.json|/etc/v2ray.confdir|g" "$(outdir)/v2ray.service" > "$(outdir)/v2ray-confdir.service"
	# remove decompressed source archive
	rm -rf $(outdir)/$(NAME)-$(VERSION)

srpm: $(outdir) spec sources
	rpmbuild --define "_sourcedir $(outdir)" \
	         --define "_srcrpmdir $(outdir)" \
	         -bs $(outdir)/$(NAME).spec

clean:
	rm -rf $(outdir)

.PHONY: all init spec sources srpm clean
