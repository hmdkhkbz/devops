# Common Role for Kubernetes Setup

## Overview

The **Common** Ansible role is designed to perform essential system configurations required for setting up a Kubernetes environment. This role ensures that all target machines are properly prepared by handling tasks such as system updates, package installations, network configurations, swap management, kernel parameter tuning, and service management.

## Features

- **Connectivity Check:** Verifies the ability to communicate with target hosts.
- **System Update & Upgrade:** Keeps the system packages up-to-date.
- **Package Installation:** Installs necessary base packages required for Kubernetes operations.
- **DNS Configuration:** Configures DNS settings using `resolvconf` and updates `/etc/resolv.conf`.
- **Swap Management:** Disables swap both temporarily and permanently to meet Kubernetes requirements.
- **Sysctl Configuration:** Sets kernel parameters essential for Kubernetes networking.
- **Kernel Modules:** Loads and ensures necessary kernel modules are available and persist across reboots.
- **NTP Service:** Installs and configures the NTP service for accurate time synchronization.
- **Hostname Configuration:** Sets the hostname of each host based on inventory data.

## Requirements

- **Ansible Version:** Ensure you have Ansible 2.9 or later installed.
- **Operating System:** Tested on Debian-based distributions (e.g., Ubuntu). Adjustments may be needed for other OS types.
- **Privileges:** The role requires elevated privileges to perform system-level configurations. Ensure that you run the playbook with appropriate privileges (e.g., using `become: yes`).

## Role Variables

The role utilizes several variables to customize its behavior. Ensure these variables are defined in your inventory, group variables, or passed as extra variables when running the playbook.

| Variable                    | Description                                         | Default                  |
| --------------------------- | --------------------------------------------------- | ------------------------ |
| `name_server1`               | Primary DNS server IP address                      | `"1.1.1.1"`              |
| `name_server2`               | Secondary DNS server IP address                    | `"8.8.8.8"`              |
| `kube_apiserver_node_port_range` | The range of ports to reserve for NodePort services | `"30000-32767"`          |
| `sysctl_file_path`           | Path to the sysctl configuration file              | `"/etc/sysctl.d/99-kubernetes.conf"` |


*Note: `name_server1` and `name_server2` must be defined to configure DNS settings correctly.*

## Dependencies

This role does not have any external dependencies. However, it assumes that the target system uses `apt` for package management and `systemd` for service management.

## Handlers

The role includes the following handlers to manage service restarts and configurations when certain tasks make changes:

- **Reload sysctl**
  - **Action:** Reloads sysctl configurations to apply new kernel parameters.
  
- **Restart networking**
  - **Action:** Restarts networking services to apply DNS configuration changes.
  
- **Reload resolvconf**
  - **Action:** Reloads resolvconf to update DNS settings.
