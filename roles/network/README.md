# Ansible Role: Network

This role manages network configuration on Ubuntu using Netplan and configures kernel parameters for unprivileged port binding.

## Features
- **Static IP Assignment**: Configure multiple interfaces with static IPs, gateways, and DNS.
- **Rootless Podman Support**: Centrally manage sysctl settings like `net.ipv4.ip_unprivileged_port_start`.
- **Validation**: Automatically validates Netplan syntax before applying changes.

## Requirements
- Ubuntu 18.04+ (Netplan support)
- `become` privileges (root access)

## Role Variables

| Variable                  | Description                                                                  | Required | Default |
| :------------------------ | :--------------------------------------------------------------------------- | :------- | :------ |
| `network_interfaces`      | List of interface configurations. See [Interface Schema](#interface-schema). | No       | `[]`    |
| `network_sysctl_settings` | Dictionary of sysctl keys and values.                                        | No       | `{}`    |
| `network_dns_servers`     | List of DNS nameserver IP addresses to write to `/etc/resolv.conf`.           | No       | `[]`    |

### Interface Schema

| Key           | Description                                       | Required |
| :------------ | :------------------------------------------------ | :------- |
| `name`        | The name of the network interface (e.g., `eth0`). | Yes      |
| `addresses`   | List of IP addresses (IPv4 or IPv6) in CIDR notation. | Yes      |
| `gateway4`    | IPv4 gateway address.                             | No       |
| `gateway6`    | IPv6 gateway address.                             | No       |
| `nameservers` | List of DNS nameserver IP addresses.              | No       |

### Interface Example
```yaml
- name: eth0
  addresses:
    - 192.168.1.10/24
    - fd00:dead:beef::10/64
  gateway4: 192.168.1.1
  gateway6: fd00:dead:beef::1
  nameservers:
    - 1.1.1.1
    - 2606:4700:4700::1111
```

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: network
      vars:
        network_sysctl_settings:
          net.ipv4.ip_unprivileged_port_start: 80
          net.ipv4.ip_forward: 1
        network_interfaces:
          - name: enp0s3
            addresses:
              - 10.0.2.15/24
            gateway4: 10.0.2.2
            nameservers:
              - 8.8.8.8
```

## Security Considerations
- Netplan configuration files are created with `0600` permissions.
- Lowering `ip_unprivileged_port_start` or enabling `ip_forward` has security implications for the host. Use the principle of least privilege.
