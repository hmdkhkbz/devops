# Ansible Role: Kubernetes Pre-Setup on Debian-based Systems

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
- [Usage](#usage)

## Overview

This Ansible role automates the installation and configuration of Kubernetes components (`kubelet`, `kubeadm`, and `kubectl`) on Debian-based systems (e.g., Debian, Ubuntu). Additionally, it manages the `/etc/hosts` file to ensure proper hostname resolution within the Kubernetes cluster.

## Features

- **System Validation:** Ensures the target system is Debian-based.
- **APT Repository Setup:** Adds the Kubernetes APT repository and its GPG key.
- **Package Installation:** Installs necessary Kubernetes packages without recommended extras.
- **Package Management:** Holds Kubernetes packages to prevent unintended upgrades.
- **Hosts Management:** Updates the `/etc/hosts` file with inventory hosts and Kubernetes load balancer addresses.

## Requirements

- **Operating Systems:** Debian, Ubuntu, or other Debian-based distributions.
- **Ansible:** Version 2.9 or higher.
- **Privileges:** Sudo privileges are required to perform system-level changes.

## Role Variables

| Variable                                   | Description                                                                                                                                                        | Default | Required |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- | -------- |
| `k8s_version`                              | Specifies the version of Kubernetes to install.                                                                                                                    | `1.31`| Yes      |
| `dns_domain`                               | The DNS domain to append to hostnames in the `/etc/hosts` file.                                                                                                   | `mycluster.local` | Yes |
| `populate_inventory_to_hosts_file`         | Boolean flag to determine whether to populate inventory hosts into the `/etc/hosts` file.                                                                          | `true`  | No       |
| `populate_loadbalancer_apiserver_to_hosts_file` | Boolean flag to determine whether to add the Kubernetes API server load balancer address to the `/etc/hosts` file.                                                  | `false` | No       |
| `apiserver_loadbalancer_domain_name`       | The domain name for the Kubernetes API server load balancer. Required if `populate_loadbalancer_apiserver_to_hosts_file` is `true`.                                | N/A     | Conditional |
| `loadbalancer_apiserver.address`           | The IP address of the Kubernetes API server load balancer. Required if `populate_loadbalancer_apiserver_to_hosts_file` is `true`.                                  | N/A     | Conditional |

### Example Variable Configuration

```yaml
k8s_version: "1.26"
dns_domain: "mycluster.local"
populate_inventory_to_hosts_file: true
populate_loadbalancer_apiserver_to_hosts_file: true
apiserver_loadbalancer_domain_name: "k8s-api.mycluster.local"
loadbalancer_apiserver:
  address: "192.168.1.100"
```

## Dependencies

This role does not have any external dependencies. It uses standard Ansible modules to perform tasks.

## Example Playbook

```yaml
---
- hosts: k8s_nodes
  become: true
  vars:
    k8s_version: "1.26"
    dns_domain: "mycluster.local"
    populate_inventory_to_hosts_file: true
    populate_loadbalancer_apiserver_to_hosts_file: true
    apiserver_loadbalancer_domain_name: "k8s-api.mycluster.local"
    loadbalancer_apiserver:
      address: "192.168.1.100"
  roles:
    - role: your_namespace.kubernetes_setup
```

Replace `your_namespace.kubernetes_setup` with the appropriate role path or namespace as per your Ansible setup.

## Usage

1. **Clone the Role:**

   ```bash
   ansible-galaxy install your_namespace.kubernetes_setup
   ```

2. **Define Inventory:**

   Ensure your inventory has a group named `k8s_cluster` with the target hosts.

   ```ini
   [k8s_cluster]
   node1 ansible_host=192.168.1.10
   node2 ansible_host=192.168.1.11
   node3 ansible_host=192.168.1.12
   ```

3. **Configure Variables:**

   Customize the variables as per your environment either in the playbook, inventory variables, or `group_vars`.

4. **Run the Playbook:**

   Execute the playbook to apply the role.

   ```bash
   ansible-playbook -i inventory.ini setup_k8s.yml
   ```

**Note:** Always ensure that the variables `apiserver_loadbalancer_domain_name` and `loadbalancer_apiserver.address` are defined when `populate_loadbalancer_apiserver_to_hosts_file` is set to `true` to avoid configuration issues.