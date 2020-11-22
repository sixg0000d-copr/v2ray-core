# v2ray-core
Build v2fly/v2ray-core on Fedora Copr

[![COPR build Status](https://copr.fedorainfracloud.org/coprs/sixg0000d/v2ray/package/v2ray-core/status_image/last_build.png)](https://copr.fedorainfracloud.org/coprs/sixg0000d/v2ray/package/v2ray-core/)

## Installation
```sh
# You may need to run the following commands with sudo
dnf copr enable sixg0000d/v2ray
dnf install v2ray-core
```

## Usage
1. Configure v2ray in:
```
/etc/v2ray/config.json
```
2. Start the service:
```sh
systemctl enable v2ray.service --now
```

## Build for your self
1. Follow the [guide](https://docs.fedoraproject.org/en-US/quick-docs/create-hello-world-rpm/#_development_environment) to install `rpmbuild`, `mock`.
2. Make srpm:
```sh
git clone https://github.com/sixg0000d-copr/v2ray-core.git
cd v2ray-core
make -f .copr/Makefile srpm
```
3. Run build on mock:
```sh
mock outdir/v2ray-core-*.srpm
```
