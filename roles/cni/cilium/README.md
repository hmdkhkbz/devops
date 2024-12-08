# Cilium Ansible Role

This Ansible role installs and configures **Cilium** and **Hubble CLI**, providing advanced networking and observability capabilities for Kubernetes clusters. The role is flexible, allowing customization of Cilium and Hubble versions, configurations, Gateway API CRDs, and optional components to tailor the deployment to your specific needs.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [Cilium CLI Installation](#cilium-cli-installation)
  - [Cilium Configuration](#cilium-configuration)
  - [Hubble CLI Installation](#hubble-cli-installation)
  - [Gateway API CRDs](#gateway-api-crds)
- [Handlers](#handlers)
- [Example Playbook](#example-playbook)
- [Template Structure](#template-structure)
- [Variable Files](#variable-files)

## Requirements

- **Ansible**: Version 2.9 or higher
- **Supported Operating Systems**:
  - **Linux-based** (e.g., Ubuntu, CentOS, Debian)
- **Dependencies**:
  - `kubectl` installed and configured on the target machines
  - Access to the Kubernetes cluster via `kubeconfig`

## Role Variables

### Basic Configuration

These variables control the fundamental settings for Cilium and Hubble, including version management and binary paths.

| Variable                       | Description                                                                                             | Default                           |
|--------------------------------|---------------------------------------------------------------------------------------------------------|-----------------------------------|
| `cilium_version`               | Version of Cilium to install, upgrade, or downgrade. If not specified, defaults to `1.14.3`.            | `1.14.3`                           |
| `cilium_cli_version`           | Version of the Cilium CLI to install. If left blank, the latest stable version is determined and installed.| `""` (auto-detect latest)         |
| `cilium_cli_bin_path`          | Path where the Cilium CLI binary will be installed.                                                    | `/usr/local/bin/cilium`           |
| `cilium_kubectl_bin_path`      | Path to the `kubectl` binary.                                                                           | `/usr/local/bin/kubectl`          |
| `cilium_kubeconfig_path`       | Path to the Kubernetes configuration file (`kubeconfig`).                                              | `~/.kube/config`                   |
| `cilium_restart_on_config_change` | Determines whether to restart Cilium components when the configuration changes.                        | `false`                            |

### Cilium CLI Installation

Variables related to the installation and architecture of the Cilium CLI.

| Variable             | Description                                                                       | Default           |
|----------------------|-----------------------------------------------------------------------------------|-------------------|
| `cilium_arch`        | Architecture for Cilium CLI (`arm64` or `amd64`). Determined automatically based on system architecture. | Automatically set  |

### Cilium Configuration

These variables define the configuration settings for Cilium, enabling customization of networking, IPAM, BPF settings, routing modes, and more.

#### Basic Settings

| Variable                       | Description                                                                                             | Default                           |
|--------------------------------|---------------------------------------------------------------------------------------------------------|-----------------------------------|
| `cilium_kube_proxy_replacement` | Enables or disables the replacement of `kube-proxy` with Cilium's BPF datapath. (`true` or `false`).     | `false`                           |
| `cilium_kube_proxy_healthz_address` | (Optional) Healthz server bind address for the kube-proxy replacement.                               | `""`                              |
| `cilium_operator_configuration` | (Optional) Configuration settings for the Cilium Operator, such as enabling it and setting replicas.    | `{}`                              |
| `cilium_k8s_service_host`      | Kubernetes service host.                                                                              | `""`                              |
| `cilium_k8s_service_port`      | Kubernetes service port.                                                                              | `6443`                            |
| `cilium_ipam_configuration`    | (Optional) IP Address Management configuration.                                                       | `{}`                              |
| `cilium_bpf_configuration`     | (Optional) BPF-related configuration settings.                                                        | `{}`                              |
| `cilium_enable_ipv4`           | Enables IPv4 support in Cilium.                                                                       | `true`                            |
| `cilium_enable_ipv6`           | Enables IPv6 support in Cilium.                                                                       | `false`                           |
| `cilium_routing_mode`          | Routing mode for Cilium (`native` or `tunnel`).                                                      | `tunnel`                          |
| `cilium_tunnel_protocol`       | Tunneling protocol used in tunnel mode (`vxlan` or `geneve`).                                       | `geneve`                          |
| `cilium_enable_ipv4_masq`      | Enables IPv4 masquerading for traffic leaving the node from endpoints.                               | `true`                            |
| `cilium_enable_ipv6_masq`      | Enables IPv6 masquerading for traffic leaving the node from endpoints.                               | `false`                           |
| `cilium_ipv4_native_cidr`      | (Optional) Specifies IPv4 CIDR for native routing.                                                   | `""`                              |
| `cilium_ipv6_native_cidr`      | (Optional) Specifies IPv6 CIDR for native routing.                                                   | `""`                              |
| `cilium_autodirect_node_routes` | (Optional) Enables automatic installation of PodCIDR routes between worker nodes.                   | `false`                           |
| `cilium_load_balancer_config`  | (Optional) Configuration for service load balancing.                                                 | `{}`                              |
| `cilium_maglev_config`         | (Optional) Configuration for Maglev consistent hashing.                                             | `{}`                              |
| `cilium_l7_proxy`              | Enables Layer 7 network policies in Cilium.                                                          | `true`                            |
| `cilium_bgp_control_plane_config` | (Optional) Configuration for BGP control plane.                                                  | `{}`                              |
| `cilium_ingress_controller_config` | (Optional) Configuration for the Cilium ingress controller.                                     | `{}`                              |
| `cilium_gateway_api_config`    | (Optional) Configuration to enable Gateway API support in Cilium.                                   | `{}`                              |
| `cilium_encryption_config`     | (Optional) Configuration for transparent network encryption.                                        | `{}`                              |
| `cilium_ip_masq_agent_config`  | (Optional) Configuration for the eBPF-based IP masquerade agent.                                    | `{}`                              |
| `cilium_rollout_pods_on_update` | Determines whether to automatically rollout Cilium agent pods when the ConfigMap is updated.        | `false`                           |
| `cilium_update_strategy`       | (Optional) Configuration for the Cilium agent update strategy (e.g., rolling updates).               | `{}`                              |
| `cilium_additional_options`    | (Optional) Additional YAML-formatted options to pass to the Cilium configuration.                     | `{}`                              |

### Hubble CLI Installation

Variables related to the installation and configuration of Hubble CLI, Cilium's observability tool.

| Variable                         | Description                                                                                                 | Default                           |
|----------------------------------|-------------------------------------------------------------------------------------------------------------|-----------------------------------|
| `cilium_install_hubble_cli`      | Determines whether to install the Hubble CLI.                                                               | `true`                            |
| `cilium_hubble_cli_version`      | Version of the Hubble CLI to install. If left blank, the latest stable version is determined and installed. | `""` (auto-detect latest)         |
| `cilium_hubble_cli_bin_path`     | Path where the Hubble CLI binary will be installed.                                                        | `/usr/local/bin/hubble`           |

### Gateway API CRDs

Variables to manage the installation of Gateway API Custom Resource Definitions (CRDs) required by Cilium.

| Variable                     | Description                                                                                             | Default       |
|------------------------------|---------------------------------------------------------------------------------------------------------|---------------|
| `cilium_install_gw_api_crds` | Determines whether to install the required Gateway API CRDs.                                            | `false`       |
| `cilium_gw_api_crds_version` | Specifies the version of Gateway API CRDs to install. Required if `cilium_install_gw_api_crds` is `true`. | `"0.7.0"`    |

## Handlers

- **Restart Cilium if Config Changed**: Restarts Cilium components (operator and daemonset) when the configuration file is updated and `cilium_restart_on_config_change` is set to `true`.

## Example Playbook

```yaml
- name: Deploy Cilium and Hubble with custom configuration
  hosts: all
  become: true
  roles:
    - role: cilium
      vars:
        cilium_version: "1.16.0"
        cilium_cli_version: "v1.16.0"
        cilium_kube_proxy_replacement: true
        cilium_enable_ipv6: true
        cilium_routing_mode: "native"
        cilium_ipv4_native_cidr: "10.244.0.0/16"
        cilium_ipv6_native_cidr: "fd00::/48"
        cilium_install_gw_api_crds: true
        cilium_gw_api_crds_version: "1.1.0"
        cilium_hubble_cli_version: "v0.15.0"
        cilium_hubble_config:
          enabled: true
          socketPath: "/var/run/cilium/hubble.sock"
          relay:
            enabled: true
          ui:
            enabled: true
```

## Template Structure

### `templates/cilium-config.j2`

This Jinja2 template generates the `cilium-config.yml` file based on the provided variables.

## Variable Files

The role includes variable files to manage different versions of Gateway API CRDs. Place these files in the `vars/` directory.

### Example Variable Files

- `vars/0.5.1_gw_api_crds.yaml`
- `vars/0.7.0_gw_api_crds.yaml`
- `vars/1.0.0_gw_api_crds.yaml`
- `vars/1.1.0_gw_api_crds.yaml`

Each file should define the `cilium_gw_api_crds` list with the appropriate CRD URLs for the specified version.

```yaml
---
cilium_gw_api_crds:
  - "https://raw.githubusercontent.com/cilium/cilium/v1.14.3/examples/kubernetes/gateway-api/standard-classes.yaml"
  - "https://raw.githubusercontent.com/cilium/cilium/v1.14.3/examples/kubernetes/gateway-api/gateway-class.yaml"
```

## Summary

This Ansible role automates the installation and configuration of Cilium and Hubble CLI within your Kubernetes clusters, ensuring a consistent and reliable networking and observability setup. By leveraging modular tasks, flexible variable configurations, and robust error handling, the role is designed for ease of use and maintainability.

