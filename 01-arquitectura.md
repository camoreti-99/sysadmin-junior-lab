# 01. Arquitectura y Red

## Segmentación de Red
- **WAN (NAT / Host):** Internet access (`ens33`) - via VMware.
- **LAN (`vmnet2`):** Subnet `10.10.10.0/24` (`ens34`) - net and client servers.
- **DMZ (`vmnet3`):** Subnet `10.10.20.0/24` (`ens35`) - isolated for public access.

(Screenshots - VMware-net-conf_1, VMware-net-conf_2, VMware-net-conf_3)

## Enrutamiento IP
Packet forwarder active in `fw01` (Screenshot - Firewall-ip-forward_1, Firewall-ip-forward_2).

## Tabla de IPs

| Máquina | SO | Interfaz | IP | Función |
| :--- | :--- | :--- | :--- | :--- |
| **fw01** | Debian 12 | `ens33` / `ens34` / `ens35` | DHCP / `10.10.10.1` / `10.10.20.1` | Firewall / Router / DHCP |
| **srv01** | Ubuntu Server 26.04 | `ens33` | `10.10.10.10` | Bind9 / Samba / Nginx - Intranet |
| **web01** | Ubuntu Server 26.04 | `ens33` | `10.10.20.10` | Nginx - DMZ |
| **mon01** | Ubuntu Server 26.04 | `ens33` | `10.10.10.20` | Prometheus / Grafana / Restic |
| **cli01** | Ubuntu Desktop 26.04 | DHCP | `10.10.10.50-200` | Linux Client |
| **win01** | Windows 11 Pro | DHCP | `10.10.10.x` | Windows Client |
