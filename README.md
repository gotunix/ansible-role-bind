# Ansible Role — Bind9 DNS Server

Manage a BIND9 nameserver on Debian/Ubuntu hosts. The role installs
`bind9`, writes configuration, and deploys forward/reverse zone files
based on variable definitions.

## Features

* Install/remove `bind9` package and ensure service is running.
* Manage `/etc/bind/named.conf.local` with zone declarations.
* Populate zone files from simple templates (records defined in vars).
* Support both forward and reverse zones.
* Variables are kept under `bind9_*` namespace.

## Requirements

- Ansible ≥ 2.9
- Debian/Ubuntu targets (package names and paths hardcoded).

## Variables

| Variable | Default | Description |
|---|---|---|
| `bind9_state` | `present` | `present` or `absent` (controls package/service) |
| `bind9_listen_addresses` | [`"127.0.0.1"`] | Addresses for `listen-on` option |
| `bind9_forwarders` | `[]` | DNS servers to forward to |
| `bind9_zones` | `[]` | List of zone definitions (see below) |

### Zone definition format

Each entry in `bind9_zones` is a dict with at least `name`, `type` and
`records`.  Example:

```yaml
bind9_zones:
  - name: example.com
    type: master
    file: "/etc/bind/zones/db.example.com"
    records:
      - { name: "@", type: SOA, value: "ns1.example.com. admin.example.com. ( 2025010101 3600 1800 604800 86400 )" }
      - { name: "@", type: NS, value: "ns1.example.com." }
      - { name: "www", type: A, value: "192.0.2.10" }
  - name: "0.168.192.in-addr.arpa"
    type: master
    file: "/etc/bind/zones/db.192.168.0.rev"
    records:
      - { name: "@", type: SOA, value: "ns1.example.com. admin.example.com. ( 2025010101 3600 1800 604800 86400 )" }
      - { name: "@", type: NS, value: "ns1.example.com." }
      - { name: "10", type: PTR, value: "www.example.com." }
```

The `file` paths are deployed via templates; directories are created as
needed.

## Example usage

```yaml
- hosts: dns
  become: true
  vars:
    bind9_listen_addresses:
      - "127.0.0.1"
      - "192.0.2.5"
    bind9_forwarders:
      - "8.8.8.8"
      - "8.8.4.4"
    bind9_zones:
      - name: example.com
        type: master
        file: "/etc/bind/zones/db.example.com"
        records:
          - { name: "@", type: SOA, value: "ns1.example.com. admin.example.com. ( 2025010101 3600 1800 604800 86400 )" }
          - { name: "@", type: NS, value: "ns1.example.com." }
          - { name: "www", type: A, value: "192.0.2.10" }
      - name: "0.168.192.in-addr.arpa"
        type: master
        file: "/etc/bind/zones/db.192.168.0.rev"
        records:
          - { name: "@", type: SOA, value: "ns1.example.com. admin.example.com. ( 2025010101 3600 1800 604800 86400 )" }
          - { name: "@", type: NS, value: "ns1.example.com." }
          - { name: "10", type: PTR, value: "www.example.com." }
  roles:
    - role: bind9
```

Tags available: `bind9`, `install`, `configure`, `uninstall`.
