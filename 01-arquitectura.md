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
flowchart LR

    %% =====================================================
    %% WAN
    %% =====================================================

    INTERNET["🌐 INTERNET<br/>VMware NAT"]

    %% =====================================================
    %% FIREWALL
    %% =====================================================

    FW["🔥 fw01 · FIREWALL / ROUTER<br/><br/>ens33 · WAN · DHCP<br/>ens34 · LAN · 10.10.10.1<br/>ens35 · DMZ · 10.10.20.1<br/><br/>nftables · NAT · dnsmasq"]

    %% =====================================================
    %% LAN
    %% =====================================================

    subgraph LAN["🏢 LAN · 10.10.10.0/24 · vmnet2"]

        SRV["🖥️ srv01<br/>10.10.10.10<br/><br/>BIND9 · :53<br/>Nginx Intranet · :80<br/>Samba · :445<br/>SSH · :22"]

        MON["📊 mon01<br/>10.10.10.20<br/><br/>Prometheus · :9090<br/>Grafana · :3000<br/>Node Exporter · :9100<br/>Restic Backup"]

        CLI["💻 cli01<br/>10.10.10.x<br/>Ubuntu Desktop<br/>DHCP"]

        WIN["💻 win01<br/>10.10.10.x<br/>Windows 11<br/>DHCP · RDP :3389"]

    end

    %% =====================================================
    %% DMZ
    %% =====================================================

    subgraph DMZ["🔒 DMZ · 10.10.20.0/24 · vmnet3"]

        WEB["🌐 web01<br/>10.10.20.10<br/><br/>Nginx Pública · :80<br/>SSH · :22<br/>Node Exporter · :9100"]

    end

    %% =====================================================
    %% HOST / VMWARE
    %% =====================================================

    HOST["🖥️ HOST / VMware<br/><br/>vmnet8 · NAT<br/>vmnet2 · LAN<br/>vmnet3 · DMZ"]

    %% =====================================================
    %% TOPOLOGÍA PRINCIPAL
    %% =====================================================

    INTERNET -->|"01 · WAN / NAT"| FW
    FW -->|"02 · LAN"| LAN
    FW -->|"03 · DMZ"| DMZ

    HOST -.->|"04 · vmnet2"| LAN
    HOST -.->|"04 · vmnet3"| DMZ

    %% =====================================================
    %% CONEXIONES INTERNAS SIMPLIFICADAS
    %% =====================================================

    CLI --- SRV
    WIN --- SRV
    MON --- SRV

    %% =====================================================
    %% LAN → DMZ
    %% =====================================================

    LAN -.->|"05 · LAN → DMZ"| FW

    %% =====================================================
    %% POLÍTICA DMZ
    %% =====================================================

    WEB -.->|"06 · DMZ → LAN<br/>BLOQUEADO"| FW

    %% =====================================================
    %% ESTILOS
    %% =====================================================

    classDef firewall fill:#ffd6d6,stroke:#b00000,stroke-width:3px,color:#111
    classDef server fill:#d9ead3,stroke:#38761d,stroke-width:2px,color:#111
    classDef monitor fill:#e4d7f5,stroke:#674ea7,stroke-width:2px,color:#111
    classDef client fill:#d9eaf7,stroke:#1155cc,stroke-width:2px,color:#111
    classDef dmz fill:#fce5cd,stroke:#e69138,stroke-width:2px,color:#111
    classDef host fill:#eeeeee,stroke:#666666,stroke-width:2px,color:#111

    class FW firewall
    class SRV server
    class MON monitor
    class CLI,WIN client
    class WEB dmz
    class HOST,INTERNET host
    class CLI01,WIN01 client
    class WEB01 dmz
    class EXT,DNS8,DNS11,NAT,HOSTPC,VMNET8,VMNET2,VMNET3 wan
    class LAN_SW,DMZ_SW switch
