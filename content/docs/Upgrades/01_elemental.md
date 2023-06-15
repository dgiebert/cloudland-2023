---
title: Elemental OS
weight: 1
---

# Elemental OS

1. Build a new version
    ```sh
    export IMAGE_VERSION=v0.0.2
    docker buildx build . \
      --push \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=${IMAGE_VERSION} \
      --tag dgiebert/rpi-os-image:${IMAGE_VERSION} \
      --target os
    ```
    ```sh
    export IMAGE_VERSION=v0.0.2
    buildah manifest create dgiebert/rpi-os-image:${IMAGE_VERSION}
    buildah build \
      --platform linux/arm64,linux/amd64 \
      --build-arg IMAGE_REPO=dgiebert/rpi-os-image \
      --build-arg IMAGE_TAG=${IMAGE_VERSION} \
      --manifest dgiebert/rpi-os-image:${IMAGE_VERSION} \
      --target os
    buildah manifest push --all "localhost/dgiebert/rpi-os-image:${IMAGE_VERSION}" "docker://docker.io/dgiebert/rpi-os-image:${IMAGE_VERSION}"
    ```
1. Create a list with the available versions in the following JSON format
    ```json
    [
      {
        "metadata": {
          "name": "v0.0.1"
        },
        "spec": {
          "version": "v0.0.1",
          "type": "container",
          "metadata": {
            "upgradeImage": "dgiebert/rpi-os-image:v0.0.1"
          }
        }
      },
      {
        "metadata": {
          "name": "v0.0.2"
        },
        "spec": {
          "version": "v0.0.2",
          "type": "container",
          "metadata": {
            "upgradeImage": "dgiebert/rpi-os-image:v0.0.2"
          }
        }
      }
    ]
    ```
2. Create and upload the needed image
    ```sh
    docker buildx build . \
      --target versions \
      --platform linux/arm64,linux/amd64 \
      --push \
      -t dgiebert/rpi-os-versions
    ```
    ```sh
    buildah manifest create dgiebert/rpi-os-versions
    buildah build \
      --platform linux/arm64,linux/amd64 \
      --target versions \
      --manifest dgiebert/rpi-os-versions
    buildah manifest push --all "localhost/dgiebert/rpi-os-versions" "docker://docker.io/dgiebert/rpi-os-versions"
    ```
3. Create a Version Channel to list available images
    ```yaml
    apiVersion: elemental.cattle.io/v1beta1
    kind: ManagedOSVersionChannel
    metadata:
      name: dgiebert-rpi-versions
      namespace: fleet-default
    spec:
      options:
        image: dgiebert/rpi-os-versions
      syncInterval: 1h
      type: custom
    ```
4. Create the Upgrade
    ```yaml
    apiVersion: elemental.cattle.io/v1beta1
    kind: ManagedOSImage
    metadata:
      name: rpi
      namespace: fleet-default
    spec:
      clusterTargets:
      - clusterName: elemental
      managedOSVersionName: v0.0.2
      osImage: ''
    ```