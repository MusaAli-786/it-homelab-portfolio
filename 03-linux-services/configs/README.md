# Configuration files

Copies of the configuration files as they run on each server, so the actual settings are visible without reading the build steps.

| Path | Installed at | Purpose |
|---|---|---|
| `dns01/50-cloud-init.yaml` | `/etc/netplan/50-cloud-init.yaml` | Static IP, gateway and DNS for `dns01` |
| `dns01/named.conf.options` | `/etc/bind/named.conf.options` | BIND global options: listen address, allowed clients, recursion |
| `dns01/named.conf.local` | `/etc/bind/named.conf.local` | Zone declarations: `lab.local` (primary) and `corp.local` (forwarded to DC01) |
| `dns01/db.lab.local` | `/etc/bind/db.lab.local` | Records for the `lab.local` zone |
| `dhcp01/50-cloud-init.yaml` | `/etc/netplan/50-cloud-init.yaml` | Static IP, gateway and DNS for `dhcp01` |
| `dhcp01/dhcpd.conf` | `/etc/dhcp/dhcpd.conf` | DHCP scope for the Workstations segment, plus an empty block for the server's own network |
| `testvm-workstation/00-installer-config.yaml` | `/etc/netplan/00-installer-config.yaml` | Test client switched from a fixed address to DHCP |

`dhcp01` also sets `INTERFACESv4="ens33"` in `/etc/default/isc-dhcp-server`, which tells the service which network card to listen on.
