MAKEFILE_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
PROJECT_ROOT := $(abspath $(MAKEFILE_DIR)/..)

spec := $(or $(spec), $(PROJECT_ROOT)/v2ray-core.spec)
outdir := $(or $(outdir), $(PROJECT_ROOT)/outdir)
name := $(shell awk '/Name:/ { print $$2 }' $(spec))
version := $(shell awk '/Version:/ { print $$2 }' $(spec))

all: srpm

$(outdir):
	[ -d "$(outdir)" ] || mkdir -p $(outdir)

ifeq ($(shell id -u),0)
init:
	dnf -y install git

	# install go 1.16.3
	curl -L https://golang.org/dl/go1.16.3.linux-amd64.tar.gz -o go1.16.3.linux-amd64.tar.gz
	rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.3.linux-amd64.tar.gz
export PATH := $(PATH):/usr/local/go/bin
else
init:
	sudo dnf -y install git golang
endif

.ONESHELL:
source: init $(outdir)
	cd $(outdir)
	curl -L https://github.com/v2fly/v2ray-core/archive/v$(version)/$(name)-$(version).tar.gz -o $(name)-$(version).tar.gz
	tar xzf $(name)-$(version).tar.gz
	cd $(name)-$(version)
	go mod vendor
	cd ..
	tar czf $(name)-$(version)-vendored.tar.gz $(name)-$(version)

	# Miscellaneous
	rm -rf $(outdir)/$(name)-$(version) $(outdir)/$(name)-$(version).tar.gz
	cp $(PROJECT_ROOT)/sources/* $(outdir)
	cp $(spec) $(outdir)

srpm: $(outdir) $(spec) source
	rpmbuild \
	--define "_sourcedir $(outdir)" \
	--define "_srcrpmdir $(outdir)" \
	-bs $(spec)

clean:
	rm -rf $(outdir)

.PHONY: all init source srpm clean
