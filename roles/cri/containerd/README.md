# Containerd Ansible Role

This Ansible role installs and configures `containerd`, a widely used container runtime, especially in Kubernetes environments. The role is flexible, allowing customization of paths, runtimes, debug settings, registry configurations, and security options.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [Debug and Metrics](#debug-and-metrics)
  - [Registry Configuration](#registry-configuration)
- [Handlers](#handlers)
- [Example Playbook](#example-playbook)
- [Template Structure](#template-structure)
- [OS-Specific Variable Files](#os-specific-variable-files)

## Requirements

- Ansible 2.9 or higher
- Supported Operating Systems:
  - **Debian-based** (e.g., Ubuntu, Debian)
  - **RHEL-based distributions** (may require customization)

## Role Variables

### Basic Configuration

These variables control basic settings for `containerd` storage, runtime, and system configurations.

| Variable                     | Description                                                                                 | Default                   |
|------------------------------|---------------------------------------------------------------------------------------------|---------------------------|
| `containerd_storage_dir`     | Directory for containerd data storage                                                       | `/var/lib/containerd`     |
| `containerd_state_dir`       | Directory for containerd state files                                                        | `/run/containerd`         |
| `containerd_default_runtime` | Default runtime for containerd                                                              | `runc`                    |
| `containerd_snapshotter`     | Snapshotter type used by containerd                                                         | `overlayfs`               |
| `containerd_oom_score`       | OOM (Out-Of-Memory) score for containerd process priority                                   | `0`                       |

### Debug and Metrics

Settings to enable logging, monitoring, and debugging capabilities in `containerd`.

| Variable                        | Description                                                      | Default       |
|---------------------------------|------------------------------------------------------------------|---------------|
| `containerd_debug_level`        | Debug level (`info`, `debug`, etc.)                              | `info`        |
| `containerd_metrics_address`    | Address for exposing containerd metrics                          | `127.0.0.1:1338` |
| `containerd_metrics_grpc_histogram` | Enable or disable gRPC histogram metrics                    | `true`        |

### Registry Configuration

Variables to configure registry mirrors, network settings, and security options.

| Variable                            | Description                                                                    | Default       |
|-------------------------------------|--------------------------------------------------------------------------------|---------------|
| `containerd_registries_mirrors`     | List of registry mirrors for pulling images                                    | `[Docker Hub]` |
| `containerd_enable_unprivileged_ports` | Allow non-root users to use ports under 1024                                 | `false`       |
| `containerd_enable_unprivileged_icmp` | Enable ICMP for non-root users                                               | `false`       |
| `containerd_enable_selinux`         | Enable SELinux support                                                         | `false`       |
| `containerd_disable_apparmor`       | Disable AppArmor (if supported by OS)                                         | `false`       |

## Handlers

- **Restart containerd**: Reloads `containerd` when configuration changes are applied.
- **Wait for containerd**: Verifies `containerd` readiness by checking the image list with `ctr`.

## Example Playbook

```yaml
- name: Deploy containerd with custom configuration
  hosts: all
  roles:
    - role: containerd
      vars:
        containerd_storage_dir: "/opt/containerd"
        containerd_debug_level: "info"
        containerd_metrics_address: "127.0.0.1:1338"
```

## Template Structure

This role includes a Jinja2 template for `config.toml`, `containerd`’s main configuration file. The template applies runtime, registry, and security settings dynamically based on variables defined in the playbook.

## OS-Specific Variable Files

The role includes OS-specific variable files (`ubuntu.yml` and `debian.yml`) to configure Docker repositories based on the detected operating system. This ensures compatibility across Debian and Ubuntu distributions.