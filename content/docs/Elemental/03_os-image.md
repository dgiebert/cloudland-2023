---
title: Custom OS Image
weight: 3
---

# Building a custom OS Image

## Preparing your machine
1. Install [docker](https://docs.docker.com/engine/install/) or [buildah](https://github.com/containers/buildah/blob/main/install.md)
1. Install emulators to build cross-platform [source](https://github.com/tonistiigi/binfmt)
    ```sh
    docker run --privileged --rm tonistiigi/binfmt --install arm64
    ```
    ```sh
    podman run --privileged --rm docker.io/tonistiigi/binfmt --install arm64
    ```



## Your first OS Image

1. Use the [wifi.connection.example](https://github.com/dgiebert/cloudland-2023/blob/master/assets/wifi.connection.example) and place it in the same folder named `wifi.connection`
1. Create a Dockerfile
    ```Dockerfile
    FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-teal/5.3:latest AS build

    # e.g check https://en.opensuse.org/Package_repositories
    RUN rpm --import http://download.opensuse.org/distribution/leap/15.4/repo/oss/gpg-pubkey-3dbdc284-53674dd4.asc && \
        zypper addrepo --refresh http://download.opensuse.org/distribution/leap/15.4/repo/oss/ oss && \
        zypper --non-interactive rm k9s kernel-firmware* && \
        zypper --non-interactive in ncdu htop && \
        # Add for Raspberry
        zypper --non-interactive in kernel-firmware-usb-network kernel-firmware-brcm kernel-firmware-bnx2 && \
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

    # Flatten the image to reduce the final size since we are not interested in layers
    FROM scratch as os
    COPY --from=build / /
    ```
2. Customize and add packages by appending to the zypper install or add complete custom Dockerfile commands 
3. Login to registry
    ```sh
    docker login
    ```
    ```sh
    buildah login docker.io
    ```
3. Export the configuration and adapt it to your username and tag version
    ```sh
    export IMAGE_REPO=dgiebert/rpi-os-image
    export IMAGE_TAG=v0.0.1
    ```
3. Build the OS Image
    ```sh
    docker buildx build . \
      --push \
      --platform linux/arm64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_TAG} \
      --tag ${IMAGE_REPO}:${IMAGE_TAG} \
      --target os
    ```
    ```sh
    buildah manifest create ${IMAGE_REPO}:${IMAGE_TAG}
    buildah build \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_TAG} \
      --manifest ${IMAGE_REPO}:${IMAGE_TAG} \
      --target os
    buildah manifest push --all "localhost/${IMAGE_REPO}:${IMAGE_TAG}" "docker://docker.io/${IMAGE_REPO}:${IMAGE_TAG}"
    ```
### Sources
- https://github.com/dgiebert/elemental-iso-builder/tree/main
- https://elemental.docs.rancher.com/customizing

