# Ansible Role: Pi-hole

An Ansible role that deploys Pi-hole as a rootless Podman container managed by systemd Quadlets, with full support for local DNS domains and DHCP management.

## Requirements

- The target system must have Podman installed (the [`podman`](../podman/) role is recommended).
- A system user dedicated to Podman must exist (the [`podman_user`](../podman_user/) role is recommended).
- To bind the privileged ports 53 (DNS) and 67 (DHCP) in rootless mode, the host's `net.ipv4.ip_unprivileged_port_start` sysctl must be set to `53` or lower (the [`network`](../network/) role can be used to set this).
- By default, the host's `systemd-resolved` service must be disabled to free up port 53. The role can handle this automatically if `pihole_disable_systemd_resolved` is set to `true`.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
| --- | --- | --- |
| `pihole_user` | **Required** | The system user that will run the rootless container. |
| `pihole_image` | `"docker.io/pihole/pihole:latest"` | The Pi-hole image to use. |
| `pihole_web_port` | `8080` | The host port to expose the lighttpd admin interface. |
| `pihole_admin_password` | `"ChangeMePlease"` | The admin password for the Pi-hole web interface. |
| `pihole_dns_servers` | `["9.9.9.9", "149.112.112.112"]` | Upstream DNS servers for Pi-hole. |
| `pihole_local_domains` | `[]` | List of local DNS mappings, each containing `domain` and `ip`. |
| `pihole_dhcp_enabled` | `false` | Whether to enable Pi-hole's built-in DHCP server. |
| `pihole_dhcp_start` | `""` | Start of the DHCP IP range. |
| `pihole_dhcp_end` | `""` | End of the DHCP IP range. |
| `pihole_dhcp_router` | `""` | Default router/gateway IP advertised to DHCP clients. |
| `pihole_dhcp_leasetime` | `"24h"` | Default DHCP lease duration. |
| `pihole_dhcp_static_leases` | `[]` | List of static DHCP mappings, each containing `mac`, `ip`, and optionally `name`. |
| `pihole_disable_systemd_resolved` | `true` | Whether to stop/disable systemd-resolved and make the host point to the local Pi-hole. |

### Local DNS Mappings Example
```yaml
pihole_local_domains:
  - domain: "router.local"
    ip: "192.168.56.1"
  - domain: "server.local"
    ip: "192.168.56.10"
```

### DHCP Configuration Example
```yaml
pihole_dhcp_enabled: true
pihole_dhcp_start: "192.168.56.100"
pihole_dhcp_end: "192.168.56.150"
pihole_dhcp_router: "192.168.56.1"
pihole_dhcp_static_leases:
  - mac: "52:54:00:12:34:56"
    ip: "192.168.56.120"
    name: "printer"
```

## Testing

This role uses Molecule and Vagrant for testing.

### Run Tests
To run the molecule test scenario:
```bash
molecule test
```
