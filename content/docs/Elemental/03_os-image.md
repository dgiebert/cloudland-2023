---
title: Custom OS Image
weight: 3
---

# Building a custom OS Image

## Preparing your machine
1. Install [docker](https://docs.docker.com/engine/install/) or [buildah](https://github.com/containers/buildah/blob/main/install.md)
1. Install emulators to build cross-platform
    ```sh
    docker run --privileged --rm tonistiigi/binfmt --install arm64
    ```
    ```sh
    podman run --privileged --rm docker.io/tonistiigi/binfmt --install arm64
    ```



## Your first OS Image

1. Create a Dockerfile
    ```Dockerfile
    FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-teal/5.3:latest AS build

    # e.g check https://en.opensuse.org/Package_repositories
    RUN rpm --import http://download.opensuse.org/distribution/leap/15.4/repo/oss/gpg-pubkey-3dbdc284-53674dd4.asc && \
        zypper addrepo --refresh http://download.opensuse.org/distribution/leap/15.4/repo/oss/ oss && \
        zypper --non-interactive rm k9s kernel-firmware* && \
        zypper --non-interactive in ncdu htop && \
        # Add for Raspberry
        zypper --non-interactive kernel-firmware-usb-network kernel-firmware-brcm kernel-firmware-bnx2 && \
        # Hack needed for Hetzner
        # zypper --non-interactive rm selinux-tools && \
        zypper --non-interactive update && \
        zypper clean --all

    COPY --chmod=0600 wifi.connection /etc/NetworkManager/system-connections/wifi.connection

    # IMPORTANT: /etc/os-release is used for versioning/upgrade. The
    # values here should reflect the tag of the image currently being built
    ARG IMAGE_REPO=norepo
    ARG IMAGE_TAG=latest
    RUN echo "IMAGE_REPO=${IMAGE_REPO}"          > /etc/os-release && \
        echo "IMAGE_TAG=${IMAGE_TAG}"           >> /etc/os-release && \
        echo "IMAGE=${IMAGE_REPO}:${IMAGE_TAG}" >> /etc/os-release
    ```
2. Customize and add packages you want to have on the nodes
3. Login to registry
    ```sh
    docker login
    ```
    ```sh
    buildah login docker.io
    ```
3. Build the OS Image
    ```sh
    docker buildx build . \
      --push \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=v0.0.1 \
      --tag dgiebert/rpi-os-image:v0.0.1 \
      --target os
    ```
    ```sh
    buildah manifest create dgiebert/rpi-os-image:v0.0.1
    buildah build \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=v0.0.1 \
      --manifest dgiebert/rpi-os-image:v0.0.1 \
      --target os
    buildah manifest push --all "localhost/dgiebert/rpi-os-image:v0.0.1" "docker://docker.io/dgiebert/rpi-os-image:v0.0.1"
    ```
### Sources
- 
- https://github.com/dgiebert/elemental-iso-builder/tree/main
- https://elemental.docs.rancher.com/customizing

