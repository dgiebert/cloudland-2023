---
title: Architecture
weight: 1
bookToc: false
---

# Architecture 

![Architecture](https://elemental.docs.rancher.com/assets/images/elemental-arch-v1.3_nobg-cbe062db521c514fc332b92ff6e7f3d5.png)

- **Elemental Operator** - connects to Rancher Manager and handles MachineRegistration and MachineInventory CRDs​
- **Elemental Register Client** - registers machines via MachineRegistrations and installs them via elemental-cli
- **Elemental CLI** - installs any elemental-toolkit based derivative. Basically an installer based on our A/B install and upgrade system
- **Elemental ISO** - includes all the tools needed to perform a full node provisioning​
- **Elemental OS image** - create a new image using a Dockerfile based on Elemental Teal image.​
- **Elemental Toolkit** - includes a set of OS utilities to enable OS management via containers. Includes dracut modules, bootloader configuration, cloud-init style configuration services, etc.​

### Sources
- https://elemental.docs.rancher.com/architecture