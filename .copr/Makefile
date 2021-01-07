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


default: srpm

init:
ifeq ($(shell id -u),0)
	dnf install -y "tar" "golang >= 1.11" "git"
else
	sudo dnf install -y "tar" "golang >= 1.11" "git"
endif

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
	cd $(outdir) && \
	curl -L $(SOURCE_URL)/archive/$(TAG)/$(NAME)-$(VERSION).tar.gz -o $(NAME)-$(VERSION).tar.gz && \
	tar -xzf $(NAME)-$(VERSION).tar.gz && \
	cd $(NAME)-$(VERSION) && \
	go mod vendor && \
	tar -czf ../vendor.tar.gz vendor && \
	cd .. && \
	rm -rf $(NAME)-$(VERSION)

sources_custom: $(outdir)
	cp $(SCM_ROOT)/*.sed          $(outdir)
	cp $(SCM_ROOT)/v2ray.sysusers $(outdir)

srpm: $(outdir) spec sources sources_custom
	rpmbuild --define "_sourcedir $(outdir)" \
	         --define "_srcrpmdir $(outdir)" \
	         -bs $(outdir)/$(NAME).spec

clean:
	rm -rf $(outdir)

.PHONY: default init spec sources sources_custom srpm clean
