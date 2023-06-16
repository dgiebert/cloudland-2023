FROM registry.opensuse.org/isv/rancher/elemental/stable/teal53/15.4/rancher/elemental-teal/5.3:latest AS build

# e.g check https://en.opensuse.org/Package_repositories
RUN rpm --import http://download.opensuse.org/distribution/leap/15.4/repo/oss/gpg-pubkey-3dbdc284-53674dd4.asc && \
    zypper addrepo --refresh http://download.opensuse.org/distribution/leap/15.4/repo/oss/ oss && \
    zypper --non-interactive rm k9s kernel-firmware* selinux-tools && \
    zypper --non-interactive in ncdu htop kernel-firmware-usb-network kernel-firmware-brcm kernel-firmware-bnx2 && \
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

FROM busybox AS versions
COPY versions.json /versions.json
CMD ["/bin/cp","/versions.json","/data/output"]
