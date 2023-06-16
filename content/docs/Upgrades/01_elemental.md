---
title: Elemental OS
weight: 1
---

# Elemental OS

1. Export the configuration and adapt it to your username and tag version
    ```sh
    export IMAGE_REPO=dgiebert/rpi-os-image
    export IMAGE_TAG=v0.0.2
    ```
1. Build a new version
    ```sh
    docker buildx build . \
      --push \
      --platform linux/arm64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_VERSION} \
      --tag ${IMAGE_REPO}:${IMAGE_VERSION} \
      --target os
    ```
    ```sh
    buildah manifest create ${IMAGE_REPO}:${IMAGE_VERSION}
    buildah build \
      --platform linux/arm64 \
      --build-arg IMAGE_REPO=${IMAGE_REPO} \
      --build-arg IMAGE_TAG=${IMAGE_VERSION} \
      --manifest ${IMAGE_REPO}:${IMAGE_VERSION} \
      --target os
    buildah manifest push --all "localhost/${IMAGE_REPO}:${IMAGE_VERSION}" "docker://docker.io/${IMAGE_REPO}:${IMAGE_VERSION}"
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
1. Append to the Dockerfile to create a multi-stage Dockerfile
  ```Dockerfile
  FROM busybox AS versions
  COPY versions.json /versions.json
  CMD ["/bin/cp","/versions.json","/data/output"]
  ```
2. Create and upload the needed image
    ```sh
    docker buildx build . \
      --target versions \
      --platform linux/arm64 \
      --push \
      -t ${IMAGE_REPO}-versions
    ```
    ```sh
    buildah manifest create ${IMAGE_REPO}-versions
    buildah build \
      --platform linux/arm64 \
      --target versions \
      --manifest ${IMAGE_REPO}-versions
    buildah manifest push --all "localhost/${IMAGE_REPO}-versions" "docker://docker.io/${IMAGE_REPO}-versions"
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
        image: dgiebert/rpi-os-image-versions
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