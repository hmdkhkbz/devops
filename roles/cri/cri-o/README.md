# CRI-O Ansible Role

This Ansible role installs and configures `CRI-O`, an OCI-compliant container runtime specifically designed for Kubernetes. The role is flexible, allowing customization of storage paths, runtime settings, logging, registry configurations, and security options.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  - [Basic Configuration](#basic-configuration)
  - [Network and Runtime](#network-and-runtime)
  - [Registry Configuration](#registry-configuration)
- [Handlers](#handlers)
- [Example Playbook](#example-playbook)
- [Template Structure](#template-structure)
- [OS-Specific Variable Files](#os-specific-variable-files)

## Requirements

- Ansible 2.9 or higher
- Supported Operating Systems:
  - **Debian-based** (e.g., Ubuntu, Debian)

## Role Variables

### Basic Configuration

These variables control basic settings for `CRI-O` storage, logging, and system paths.

| Variable                     | Description                                                                                 | Default                   |
|------------------------------|---------------------------------------------------------------------------------------------|---------------------------|
| `crio_version`               | Version of CRI-O to install.                                                                | `v1.31`                   |
| `crio_log_level`             | Log level for CRI-O logging (`info`, `debug`, etc.)                                         | `info`                    |
| `crio_root`                  | Directory for CRI-O data storage                                                            | `/var/lib/containers/storage` |
| `crio_runroot`               | Directory for CRI-O runtime files                                                           | `/var/run/containers/storage` |
| `crio_storage_driver`        | Storage driver for CRI-O                                                                    | `overlay`                 |
| `crio_storage_option`        | Additional storage options                                                                  | `["overlay.mount_program=/usr/bin/fuse-overlayfs"]` |

### Network and Runtime

Settings to configure CRI-O runtimes and network plugin directories.

| Variable                           | Description                                                               | Default                   |
|------------------------------------|---------------------------------------------------------------------------|---------------------------|
| `crio_default_runtime`             | Default runtime for CRI-O                                                 | `crun`                    |
| `crio_container_exits_dir`         | Directory for CRI-O container exit files                                  | `/var/run/crio/exits`     |
| `crio_container_attach_socket_dir` | Directory for CRI-O attach sockets                                        | `/var/run/crio`           |
| `crio_network_dir`                 | Network configuration directory for CNI plugins                           | `/etc/cni/net.d/`         |
| `crio_plugin_dirs`                 | Directories where CNI plugin binaries are located                         | `["/usr/libexec/cni", "/opt/cni/bin"]` |

### Registry Configuration

Variables to configure CRI-O registry sources and options.

| Variable                        | Description                                                                 | Default                  |
|---------------------------------|-----------------------------------------------------------------------------|--------------------------|
| `crio_registries`               | List of registry configurations including prefix, security, and mirrors     | See example below        |

#### Example `crio_registries` Configuration

You can define multiple registries with options like `prefix`, `insecure`, `blocked`, and `mirrors`.

```yaml
crio_registries:
  - prefix: "docker.io"
    insecure: false
    blocked: false
    location: "registry-1.docker.io"
  - prefix: "quay.io"
    insecure: false
    blocked: false
    location: "quay.io"
```

## Handlers

- **Restart CRI-O**: Reloads and restarts CRI-O when configuration changes are applied.

## Example Playbook

```yaml
- name: Deploy CRI-O with custom configuration
  hosts: all
  roles:
    - role: crio
      vars:
        crio_log_level: "debug"
        crio_root: "/opt/crio-storage"
        crio_registries:
          - prefix: "docker.io"
            insecure: false
            blocked: false
            location: "registry-1.docker.io"
          - prefix: "quay.io"
            insecure: false
            blocked: false
            location: "quay.io"
```

## Template Structure

This role includes Jinja2 templates:
- **`crio.conf.j2`**: Configures CRI-O’s main settings including log level, storage, and runtime options.
- **`registry.conf.j2`**: Configures registries for CRI-O, including support for multiple registry mirrors.
