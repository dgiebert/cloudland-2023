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
            ssh_authorized_keys:
              - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOY5nEt0qssNTouZzN4LPg8M3OyDAwGDDvreTUMA6hQ5
      elemental:
        install:
          poweroff: true
          device: /dev/mmcblk0
          disable-boot-entry: true
        registration:
          emulate-tpm: true
          emulated-tpm-seed: -1
    machineInventoryLabels:
      manufacturer: "${System Information/Manufacturer}"
      productName: "${System Information/Product Name}"
      serialNumber: "${System Information/Serial Number}"
      machineUUID: "${System Information/UUID}"
    ```
2. Click create and "Download the Configuration File"
3. Create a folder in `rpi-image` named `overlay` and place the configuration file as `livecd-cloud-config.yaml`