---
weight: 1
title: "Preparing the node"
bookToc: false
---

# Installing Rancher
Requirements: 2 vCPU and 4GB of RAM

## cloud-init
Tested with Ubuntu 22.04 and openSUSE Leap 15.4
```yaml
#cloud-config
### System
locale: en_US.UTF-8
timezone: Europe/Berlin
### Users
user: rancher
ssh_authorized_keys:
- ssh-ed25519 .....
### Install
package_update: true
package_upgrade: true
package_reboot_if_required: true
packages: ["bash-completion", "fzf"]
### Files
write_files:
- path: /root/.bashrc
  content: |
    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
    alias k=kubectl
    complete -o default -F __start_kubectl k
    source <(kubectl completion bash | sed 's#"${requestComp}" 2>/dev/null#"${requestComp}" 2>/dev/null | head -n -1 | fzf  --multi=0 #g')
  append: true
- path: /etc/sysctl.d/90-kubelet.conf
  content: |
    vm.panic_on_oom=0
    vm.overcommit_memory=1
    kernel.panic=10
    kernel.panic_on_oops=1
    kernel.keys.root_maxbytes=25000000    
- path: /etc/sysctl.d/90-networking.conf
  content: |
    net.ipv4.conf.all.forwarding = 1
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    net.ipv6.conf.lo.disable_ipv6 = 1
- path: /etc/rancher/k3s/config.yaml
  content: |
    protect-kernel-defaults: true
```

### Sources
- https://docs.k3s.io/installation/requirements
- https://ranchermanager.docs.rancher.com/pages-for-subheaders/installation-requirements
- https://docs.k3s.io/security/hardening-guide