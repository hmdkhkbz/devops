# HAProxy Role

This Ansible role sets up an HAProxy load balancer using a Docker container. It configures HAProxy to load balance traffic for a Kubernetes cluster, including API server requests and HTTP/HTTPS traffic for worker nodes.

## Features

- Installs Docker and required dependencies for HAProxy.
- Configures HAProxy with custom settings based on the inventory structure.
- Uses a template (`haproxy.cfg.j2`) to dynamically configure backend servers (masters and workers) and frontend endpoints.
- Supports automatic restart of the HAProxy container when the configuration is updated.
- Optionally includes a HAProxy Prometheus exporter.

## Requirements

- Ansible 2.9 or later
- Docker must be supported on the target system
- Inventory setup with `masters` and `workers` groups for dynamic backend generation

## Role Variables

| Variable         | Description                                                                                   | Default |
|------------------|-----------------------------------------------------------------------------------------------|---------|
| `enable_haproxy` | Controls whether HAProxy configuration is applied. Set to `"true"` to enable.                 | `"false"` |
| `groups['masters']`  | Inventory group for Kubernetes master nodes. Used to configure the `apiservers` backend. | None |
| `groups['workers']`  | Inventory group for Kubernetes worker nodes. Used for HTTP and HTTPS backends.           | None |

## Template

The `haproxy.cfg.j2` template dynamically configures HAProxy backends and frontends based on the groups defined in the inventory:

- **apiservers** frontend listens on port 6443 and routes to `master_servers` backend.
- **http** frontend listens on port 80 and routes to `http80` backend (worker nodes).
- **https** frontend listens on port 443 and routes to `https443` backend (worker nodes).

## Usage

### Step 1: Define Inventory

Ensure your inventory has `masters` and `workers` groups for HAProxy to load balance traffic correctly. Here’s an example structure:

### Step 2: Set Variables

In your playbook or group variables, set `enable_haproxy` to `"true"` to enable HAProxy configuration:

```yaml
# inventories/group_vars/all.yml
enable_haproxy: "true"
```

### Step 3: Use the Role in a Playbook

Create a playbook that includes the HAProxy role:

```yaml
- name: Setup HAProxy Load Balancer
  hosts: haproxy
  become: true
  roles:
    - haproxy
```

## Handlers

The role includes a handler to restart the HAProxy container if the configuration is updated. This ensures that changes are applied without manual intervention.
