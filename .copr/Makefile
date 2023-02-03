MAKEFILE_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
PROJECT_ROOT := $(abspath $(MAKEFILE_DIR)/..)

spec ?= $(PROJECT_ROOT)/v2ray-core.spec
outdir ?= $(PROJECT_ROOT)/outdir
source0 = $(shell spectool -s 0 $(spec) | awk '/Source0:/ { print $$2 }')
name-version = $(shell basename -s .tar.gz $(source0))
go-version = go1.18.10
os = $(shell uname -s | tr A-Z a-z)
arch = $(shell uname -m | sed -e 's/aarch64/arm64/' -e 's/x86_64/amd64/')

all: srpm

$(outdir):
	[ -d "$(outdir)" ] || mkdir -p $(outdir)

ifeq ($(shell id -u),0)
init:
	dnf -y install git rpmdevtools

	# install $(go-version)
	curl -L https://golang.org/dl/$(go-version).$(os)-$(arch).tar.gz -o $(go-version).$(os)-$(arch).tar.gz
	rm -rf /usr/local/go && tar -C /usr/local -xzf $(go-version).$(os)-$(arch).tar.gz
export PATH := $(PATH):/usr/local/go/bin
else
init:
	sudo dnf -y install git rpmdevtools
	sudo dnf -y install "golang = 1.18.10"
endif

.ONESHELL:
source: init $(outdir)
	cd $(outdir)
	spectool -g -s 0 $(spec)
	tar xzf $(name-version).tar.gz
	cd $(name-version)
	go mod vendor
	tar czf ../$(name-version)-vendor.tar.gz vendor
	cd ..

	# Miscellaneous
	rm -rf $(outdir)/$(name-version)
	cp $(PROJECT_ROOT)/sources/* $(outdir)
	cp $(spec) $(outdir)

srpm: $(outdir) $(spec) source
	rpmbuild \
	--define "_sourcedir $(outdir)" \
	--define "_srcrpmdir $(outdir)" \
	-bs $(spec)

rpm: $(outdir) $(spec) source
	rpmbuild \
	--define "_sourcedir $(outdir)" \
	--define "_rpmdir $(outdir)" \
	-bb $(spec)

clean:
	rm -rf $(outdir)

.PHONY: all init source srpm rpm clean
