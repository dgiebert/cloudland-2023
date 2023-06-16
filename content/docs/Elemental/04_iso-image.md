---
title: Building the ISO
weight: 4
---

# Building a the ISO

> **Note**
> Make sure to check [the requirements](/docs/elemental/02_os-image/)

## Create a multi-stage
1. Adapt the `wifi.connection.example` and copy it to `wifi.connection`
1. Append to the Dockerfile to create a multi-stage Dockerfile
    ```Dockerfile
    FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-builder-image/5.3:latest AS builder
    ARG TARGETARCH
    WORKDIR /iso
    COPY --from=os / rootfs
    COPY overlay overlay

    # Fix needed for buildah
    RUN rm -rf rootfs/etc/resolv.conf && \
        ln -s /var/run/netconfig/resolv.conf rootfs/etc/resolv.conf && \
        mkdir output && \
        elemental build-iso \
            dir:rootfs \
            --bootloader-in-rootfs \
            --squash-no-compression \
            -n "elemental-teal-${TARGETARCH}" && \
        xorriso -indev "elemental-teal-${TARGETARCH}.iso" -outdev "output/elemental-teal-${TARGETARCH}.iso" -map overlay / -boot_image any replay

    FROM scratch AS iso
    COPY --from=builder /iso/output .
    ```
3. Export the configuration and adapt it to your username and tag version
    ```sh
    export IMAGE_REPO=dgiebert/rpi-os-image
    export IMAGE_TAG=v0.0.1
    ```
2. Build the ISO
    ```sh
    docker buildx build . \
      --platform linux/arm64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_TAG} \
      --target iso \
      --output .
    ```
    ```sh
    buildah build \
      --platform linux/arm64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_TAG} \
      --target iso \
      --output .
    ```
3. For the Raspberry Pi the image now needs to be modified please use the provided [rpi.sh](/assets/rpi.sh) script in the folder holding the `elemental-teal-arm64.iso`
4. Write the resulting `elemental-teal-rpi.iso` to the USB key
    ```sh
    sudo dd if=elemental-teal-rpi.iso of=/dev/sda status=progress
    ```

### Sources
- https://github.com/dgiebert/elemental-iso-builder/tree/main
- https://elemental.docs.rancher.com/customizing

