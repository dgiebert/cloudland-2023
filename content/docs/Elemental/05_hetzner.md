---
title: Running on Hetzner
weight: 4
---

# Running on Hetzner

1. Create a registration with the following configuration
    ```yaml
    config:
      cloud-config:
        users:
          - name: root
            ssh_authorized_keys:
              - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOY5nEt0qssNTouZzN4LPg8M3OyDAwGDDvreTUMA6hQ5
      elemental:
        install:
          device: /dev/sda
          system-uri: dgiebert/hetzner-os-image:v0.0.1
        registration:
          emulate-tpm: true
          emulated-tpm-seed: -1
    machineInventoryLabels:
      registrationEndpoint: rpi
      manufacturer: "${System Information/Manufacturer}"
      productName: "${System Information/Product Name}"
      serialNumber: "${System Information/Serial Number}"
      machineUUID: "${System Information/UUID}"
    ```
1. Put the vServer or Server into rescue mode
2. SSH and execute the commands to register the nodes
    ```sh
    # Install dependencies
    apt update 
    apt install -y golang libssl-dev
    # Get the Elemental CLI
    wget https://github.com/rancher/elemental-cli/releases/download/v0.3.1/elemental-v0.3.1-Linux-arm64.tar.gz
    tar -xf elemental-v0.3.1-Linux-arm64.tar.gz
    mv elemental /usr/local/bin
    # Create the needed folders, binaries
    ln -s /usr/bin/grub-editenv /usr/bin/grub2-editenv
    mkdir /oem
    # Build the elemental-register commands
    git clone https://github.com/rancher/elemental-operator.git
    cd elemental-operator/
    make register
    # Register and reboot
    ./build/elemental-register --registration-url https://cloudland.giebert.dev/elemental/registration/vg9f84spkxsm2vxrp7ngwb9dl5lqllzlw8qq48xkpk78dmbs8kwsxz --emulate-tpm true --emulated-tpm-seed -1
    reboot
    ```