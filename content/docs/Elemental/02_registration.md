---
title: Registration Endpoint
weight: 2
---

# Create the Registration Endpoints
![Registration Endpoint](/registration_endpoint.png)

1. Populate the Cloud Configuration with this example
    ```yaml
    config:
      cloud-config:
        users:
          - name: root
            passwd: rancher
      elemental:
        install:
          reboot: true
          device: /dev/mmcblk0
          debug: true
          disable-boot-entry: true
        registration:
          emulate-tpm: true
    machineInventoryLabels:
      manufacturer: "${System Information/Manufacturer}"
      productName: "${System Information/Product Name}"
      serialNumber: "${System Information/Serial Number}"
      machineUUID: "${System Information/UUID}"
    ```
2. Click create and "Download the Configuration File"
3. Create a folder in `rpi-image` named `overlay` and place the configuration file as `livecd-cloud-config.yaml`