---
title: Building the ISO
weight: 4
bookToc: false
---

# Building a the ISO

> **Note**
> Make sure to check [the requirements](/docs/elemental/02_os-image/)

## Create a multi-stage 
1. Create a multi-stage a Dockerfile or use the provided [example](/assets/Dockerfile)
    ```Dockerfile
    FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-teal/5.3:latest AS os

    # e.g check https://en.opensuse.org/Package_repositories
    RUN rpm --import http://download.opensuse.org/distribution/leap/15.4/repo/oss/gpg-pubkey-3dbdc284-53674dd4.asc && \
        zypper addrepo --refresh http://download.opensuse.org/distribution/leap/15.4/repo/oss/ oss && \
        zypper --non-interactive in ncdu htop && \
        zypper clean --all

    # IMPORTANT: /etc/os-release is used for versioning/upgrade.
    ARG IMAGE_REPO=norepo
    ARG IMAGE_TAG=latest
    RUN echo "IMAGE_REPO=${IMAGE_REPO}"          > /etc/os-release && \
        echo "IMAGE_TAG=${IMAGE_TAG}"           >> /etc/os-release && \
        echo "IMAGE=${IMAGE_REPO}:${IMAGE_TAG}" >> /etc/os-release
    
    FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-builder-image/5.3:latest AS builder
    ARG TARGETARCH
    WORKDIR /iso
    COPY --from=os / rootfs

    RUN --mount=type=bind,source=./,target=/output,rw \
        elemental build-iso \
            dir:rootfs \
            --bootloader-in-rootfs \
            --squash-no-compression \
            -o /output -n "elemental-teal-${TARGETARCH}" && \
        xorriso -indev "elemental-teal-${TARGETARCH}.iso" -outdev "output/elemental-teal-${TARGETARCH}.iso" -map overlay / -boot_image any replay

    FROM scratch AS export
    COPY --from=builder /iso/output .
    ```
2. Build the ISO
    ```sh
    docker buildx build . \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=v0.0.1 \
      --output .
    ```
    ```sh
    buildah build \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=v0.0.1 \
      --output .
    ```
3. For the Raspberry Pi the image now needs to be modified please use the provided [rpi.sh](/assets/rpi.sh) script in the folder holding the `elemental-teal-arm64.iso`
4. Write the resulting `elemental-teal-rpi.iso` to the USB key

### Sources
- https://github.com/dgiebert/elemental-iso-builder/tree/main
- https://elemental.docs.rancher.com/customizing

