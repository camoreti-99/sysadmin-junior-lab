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


# 🛡️ Network Architecture and Service Map

> [!NOTE]
> Multi-tier infrastructure divided into three network zones in `fw01` firewall, a private LAN network (`10.10.10.0/24`), a demilitarized zone DMZ (`10.10.20.0/24`) and a WAN exit.

---

## 📐 Network Topology and Flow Diagram

```mermaid
graph TD
    subgraph WAN ["🌐 ZONA WAN (Internet)"]
        EXT["Internet / NAT VMware"]
    end

    subgraph FW ["🔥 FIREWALL / ROUTER (fw01)"]
        FW_WAN["ens33 (DHCP WAN)"]
        FW_LAN["ens34 (10.10.10.1)"]
        FW_DMZ["ens35 (10.10.20.1)"]
    end

    subgraph LAN ["🏢 ZONA LAN (10.10.10.0/24) - vmnet2"]
        SRV01["🖥️ srv01 (10.10.10.10)<br/>• BIND9 DNS (53)<br/>• Nginx Intranet (80)<br/>• Samba SMB (445)"]
        MON01["📊 mon01 (10.10.10.20)<br/>• Prometheus (9090)<br/>• Grafana (3000)<br/>• Restic Backup"]
        CLI01["💻 cli01 (10.10.10.x)<br/>Ubuntu Desktop"]
        WIN01["💻 win01 (10.10.10.x)<br/>Windows 11 (RDP 3389)"]
    end

    subgraph DMZ ["🔒 ZONA DMZ (10.10.20.0/24) - vmnet3"]
        WEB01["🌐 web01 (10.10.20.10)<br/>• Nginx Pública (80)<br/>• Node Exporter (9100)"]
    end

    %% Conexiones
    EXT <==>|NAT / Masquerade| FW_WAN
    FW_LAN <==> LAN
    FW_DMZ <==> DMZ

    CLI01 -.->|HTTP:80 / SMB:445| SRV01
    WIN01 -.->|HTTP:80 / SMB:445| SRV01
    CLI01 ==>|HTTP:80 (nftables)| WEB01
    MON01 ==>|Scrape:9100| WEB01
    MON01 -.->|SSH:22 Backup| SRV01