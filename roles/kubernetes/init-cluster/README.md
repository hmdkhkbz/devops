# Ansible Role: Kubernetes Cluster Init

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
- [Usage](#usage)

## Overview

This Ansible role automates the setup of the Kubernetes Cluster Init nodes using `kubeadm`. It handles tasks such as initializing the first control plane node, joining additional master nodes and workers, pulling required Kubernetes images, and setting up the necessary configuration files for Kubernetes (`kubeconfig`).

The role supports the following actions:
- Checking if `kubeadm` has already been executed.
- Initializing the first control plane.
- Adding additional master and worker nodes to the cluster.
- Configuring Kubernetes and copying configuration files to the appropriate directories.

## Features

- **System Validation:** Ensures the first control plane node is initialized before continuing.
- **Kubernetes Initialization:** Initializes the first master node and configures the Kubernetes control plane.
- **Join Additional Masters:** Generates the join command and joins additional master nodes to the cluster.
- **Kubernetes Configuration:** Copies the Kubernetes configuration file (`kubeconfig`) to the appropriate user directories for cluster management.
- **Cross-Node Configuration:** Uses `delegate_to` to ensure tasks are run on the correct nodes (e.g., `master-1`).

## Requirements

- **Operating Systems:** Any system that supports Kubernetes, including Ubuntu, Debian, CentOS, or RHEL-based distributions.
- **Ansible:** Version 2.9 or higher.
- **Kubernetes:** `kubeadm` and `kubelet` installed.
- **Privileges:** Sudo privileges are required to perform system-level changes and execute Kubernetes-related tasks.
- **Nodes:** The first master node (`master-1`) must be manually set up before this role is applied.

## Role Variables

| Variable                                   | Description                                                                                                           | Default Value     | Required |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|-------------------|----------|
| `pod_network_cidr`                         | The CIDR block for the pod network.                                                                                   | `10.244.0.0/16`   | No       |
| `loadbalancer`                             | The hostname or IP address of the load balancer (for control plane endpoint).                                          | N/A               | Yes      |

### Example Variable Configuration

```yaml
pod_network_cidr: "10.244.0.0/16"
loadbalancer: "k8s.mycluster.local"
```

## Dependencies

This role has the following dependencies that need to be executed prior to using the **control-plane** role:

- **common role**: Sets up common configurations across all nodes, including time synchronization, disabling swap, network configurations and ensuring required kernel modules are loaded.
- **cri role**: Installs and configures the Container Runtime Interface (CRI) such as Cri-o or Containerd, which is required for Kubernetes to run containers.
- **pre-install role**: Install Kubernetes necessary packages.

Ensure these roles are executed before applying the **control-plane** role to avoid missing configuration steps.

This role does not have any other external dependencies and uses standard Ansible modules to perform tasks. However, it assumes that the target nodes are already Kubernetes-compatible, with `kubeadm` and the necessary runtime installed.

## Example Playbook

```yaml
---
- name: Set up Kubernetes Control Plane
  hosts: k8s_cluster
  become: true
  vars:
    pod_network_cidr: "10.244.0.0/16"
    loadbalancer: "k8s.mycluster.local"
  roles:
    - control-plane
```

## Usage

1. **Define Inventory:**

   Ensure your inventory has the correct host group for the control plane nodes.

   ```ini
   [k8s_cluster]
   master-1 ansible_host=192.168.1.10
   master-2 ansible_host=192.168.1.11
   master-3 ansible_host=192.168.1.12
   ```

2. **Configure Variables:**

   Customize the variables such as `loadbalancer`, `pod_network_cidr` as per your environment either in the playbook, inventory variables, or `group_vars`.


### Notes:

- The role assumes that `master-1` is already initialized and running Kubernetes. The other masters will join the cluster automatically.
- Ensure that the `loadbalancer` variable are correctly defined in your inventory or playbook to avoid configuration issues.