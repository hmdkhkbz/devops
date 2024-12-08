# Flannel Ansible Role

This Ansible role installs and configures **Flannel** with optional Wireguard encryption, providing a simple and efficient networking solution for Kubernetes clusters. The role ensures that Flannel is properly set up with the necessary RBAC permissions and DaemonSet configurations, allowing for secure and scalable network communication within your cluster.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [Flannel RBAC Configuration](#flannel-rbac-configuration)
  - [Flannel DaemonSet Configuration](#flannel-daemonset-configuration)
  - [Wireguard Encryption](#wireguard-encryption)
- [Handlers](#handlers)
- [Example Playbook](#example-playbook)
- [Template Structure](#template-structure)


## Requirements

- **Ansible**: Version 2.9 or higher
- **Supported Operating Systems**:
  - **Linux-based** (e.g., Ubuntu, CentOS, Debian)
- **Dependencies**:
  - `kubectl` installed and configured on the target machines
  - Access to the Kubernetes cluster via `kubeconfig`

## Role Variables

### Basic Configuration

These variables control the fundamental settings for Flannel, including network configurations and backend types.

| Variable                      | Description                                                                                                   | Default          |
|-------------------------------|---------------------------------------------------------------------------------------------------------------|------------------|
| `flannel_namespace`           | Namespace where Flannel components will be deployed.                                                         | `kube-system`    |
| `flannel_backend_type`        | The backend type for Flannel (e.g., `vxlan`, `wireguard`).                                                    | `vxlan`          |
| `flannel_network_cidr`        | The CIDR range for the Flannel network.                                                                       | `10.244.0.0/16`   |
| `flannel_cni_network_name`    | The name of the CNI network configuration.                                                                    | `cni0`           |
| `flannel_cni_version`         | The CNI version to use in the Flannel configuration.                                                          | `0.3.1`           |
| `flannel_enable_nftables`     | Enable nftables support in Flannel.                                                                            | `false`          |
| `flannel_update_max_unavailable` | Maximum number of Flannel pods that can be unavailable during updates.                                       | `20%`            |
| `kube_config_dir`             | Directory where Kubernetes configuration files are stored.                                                     | `/etc/kubernetes` |

### Flannel RBAC Configuration

These variables define the RBAC (Role-Based Access Control) settings required for Flannel to operate within the Kubernetes cluster.

| Variable                      | Description                                                                                                   | Default          |
|-------------------------------|---------------------------------------------------------------------------------------------------------------|------------------|
| `flannel_service_account`     | The name of the ServiceAccount for Flannel.                                                                   | `flannel`        |
| `flannel_cluster_role`        | The name of the ClusterRole for Flannel.                                                                      | `flannel-role`   |
| `flannel_cluster_role_binding` | The name of the ClusterRoleBinding for Flannel.                                                               | `flannel-role-binding` |

### Flannel DaemonSet Configuration

These variables configure the Flannel DaemonSet, including container images, resource allocations, and network settings.

| Variable                      | Description                                                                                                   | Default                          |
|-------------------------------|---------------------------------------------------------------------------------------------------------------|----------------------------------|
| `flannel_architectures`       | List of architectures to deploy Flannel DaemonSets for (e.g., `amd64`, `arm64`).                             | `['amd64']`                      |
| `flannel_cni_plugin_image`    | Docker image for the Flannel CNI plugin.                                                                      | `quay.io/flannel/cni-plugin:v1.1.0` |
| `flannel_image_repo`          | Repository for the Flannel container image.                                                                    | `quay.io/flannel/flannel`         |
| `flannel_image_tag`           | Tag for the Flannel container image.                                                                            | `v1.1.0`                          |
| `flannel_image_pull_policy`   | Image pull policy for Flannel containers.                                                                       | `IfNotPresent`                    |
| `flannel_interface`           | (Optional) Network interface to use for Flannel.                                                               | `undefined`                      |
| `flannel_interface_regexp`    | (Optional) Regular expression to match network interfaces for Flannel.                                       | `undefined`                      |
| `flannel_cpu_requests`        | CPU requests for the Flannel container.                                                                        | `100m`                            |
| `flannel_memory_requests`     | Memory requests for the Flannel container.                                                                     | `128Mi`                           |
| `flannel_cpu_limit`           | CPU limits for the Flannel container.                                                                          | `200m`                            |
| `flannel_memory_limit`        | Memory limits for the Flannel container.                                                                       | `256Mi`                           |
| `flannel_update_max_unavailable` | Maximum number of Flannel pods that can be unavailable during updates.                                       | `20%`                             |

### Wireguard Encryption

These variables enable and configure Wireguard encryption for Flannel, ensuring secure network communication.

| Variable                      | Description                                                                                                   | Default          |
|-------------------------------|---------------------------------------------------------------------------------------------------------------|------------------|
| `ignore_assert_errors`        | Ignore assertion errors during kernel version validation.                                                     | `false`          |

## Handlers

- **Restart Flannel DaemonSet**: Restarts the Flannel DaemonSet when configuration files are updated.

## Example Playbook

```yaml
- name: Deploy Flannel with Wireguard encryption
  hosts: all
  become: true
  roles:
    - role: flannel
      vars:
        flannel_namespace: "kube-system"
        flannel_backend_type: "wireguard"
        flannel_network_cidr: "10.244.0.0/16"
        flannel_cni_network_name: "cni0"
        flannel_cni_version: "0.3.1"
        flannel_enable_nftables: true
        flannel_architectures:
          - amd64
          - arm64
        flannel_service_account: "flannel-sa"
        flannel_cluster_role: "flannel-cluster-role"
        flannel_cluster_role_binding: "flannel-cluster-role-binding"
        flannel_cni_plugin_image: "quay.io/flannel/cni-plugin:v1.1.0"
        flannel_image_repo: "quay.io/flannel/flannel"
        flannel_image_tag: "v1.1.0"
        flannel_image_pull_policy: "IfNotPresent"
        flannel_cpu_requests: "100m"
        flannel_memory_requests: "128Mi"
        flannel_cpu_limit: "200m"
        flannel_memory_limit: "256Mi"
        flannel_update_max_unavailable: "20%"
        ignore_assert_errors: false
```

## Template Structure

### `templates/cni-flannel.yml.j2`

This Jinja2 template generates the `cni-flannel.yml` ConfigMap and DaemonSet manifests based on the provided variables. It configures the CNI settings, network CIDR, backend type, and optional Wireguard encryption parameters. The DaemonSet is templated for each specified architecture, ensuring compatibility across different node types.

### `templates/cni-flannel-rbac.yml.j2`

This Jinja2 template generates the RBAC manifests required for Flannel to operate within the Kubernetes cluster. It includes the Namespace, ServiceAccount, ClusterRole, and ClusterRoleBinding configurations, ensuring that Flannel has the necessary permissions to manage network resources.

## Summary

This Ansible role automates the installation and configuration of Flannel with optional Wireguard encryption within your Kubernetes clusters, ensuring a secure and efficient networking setup. By leveraging modular tasks, flexible variable configurations, and robust error handling, the role is designed for ease of use and maintainability.
