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

    %% =========================================================
    %% WAN
    %% =========================================================
    subgraph WAN ["🌐 ZONA WAN / INTERNET"]
        EXT["🌍 Internet"]
        DNS8["DNS público<br/>8.8.8.8"]
        DNS11["DNS público<br/>1.1.1.1"]
        NAT["VMware NAT"]
    end

    %% =========================================================
    %% HOST FISICO / VMWARE
    %% =========================================================
    subgraph HOST ["🖥️ HOST FÍSICO / VMWARE WORKSTATION"]
        HOSTPC["Host Windows/Linux<br/>VMware Workstation Pro"]
        VMNET8["vmnet8<br/>NAT"]
        VMNET2["vmnet2<br/>Host-only<br/>10.10.10.0/24"]
        VMNET3["vmnet3<br/>Host-only<br/>10.10.20.0/24"]
    end

    %% =========================================================
    %% FIREWALL
    %% =========================================================
    subgraph FW ["🔥 FIREWALL / ROUTER — fw01"]
        FW_WAN["ens33<br/>WAN / DHCP VMware NAT"]
        FW_LAN["ens34<br/>10.10.10.1/24<br/>Gateway LAN"]
        FW_DMZ["ens35<br/>10.10.20.1/24<br/>Gateway DMZ"]

        FW_ROUTE["IP Forwarding<br/>net.ipv4.ip_forward=1"]
        NFT["nftables<br/>Firewall + NAT"]
        DNSMASQ["dnsmasq<br/>DHCP + DNS Forwarder"]
    end

    %% =========================================================
    %% LAN
    %% =========================================================
    subgraph LAN ["🏢 ZONA LAN — 10.10.10.0/24 / vmnet2"]
        LAN_SW["🔀 LAN / vmnet2"]

        SRV01["🖥️ srv01<br/>10.10.10.10<br/><br/>BIND9 DNS :53<br/>Nginx Intranet :80<br/>Samba SMB :445<br/>SSH :22<br/>Node Exporter :9100"]

        MON01["📊 mon01<br/>10.10.10.20<br/><br/>Prometheus :9090<br/>Grafana :3000<br/>Node Exporter :9100<br/>Restic Backup"]

        CLI01["💻 cli01<br/>10.10.10.50-200<br/><br/>Ubuntu Desktop"]

        WIN01["💻 win01<br/>10.10.10.x<br/><br/>Windows 11<br/>SMB Client<br/>RDP :3389"]
    end

    %% =========================================================
    %% DMZ
    %% =========================================================
    subgraph DMZ ["🔒 ZONA DMZ — 10.10.20.0/24 / vmnet3"]
        DMZ_SW["🔀 DMZ / vmnet3"]

        WEB01["🌐 web01<br/>10.10.20.10<br/><br/>Nginx Pública :80<br/>SSH :22<br/>Node Exporter :9100"]
    end

    %% =========================================================
    %% CONEXIONES FISICAS / VIRTUALES
    %% =========================================================

    HOSTPC --- VMNET8
    HOSTPC --- VMNET2
    HOSTPC --- VMNET3

    VMNET8 --- NAT
    NAT --- FW_WAN
    EXT --- NAT

    VMNET2 --- LAN_SW
    VMNET3 --- DMZ_SW

    LAN_SW --- FW_LAN
    DMZ_SW --- FW_DMZ

    LAN_SW --- SRV01
    LAN_SW --- MON01
    LAN_SW --- CLI01
    LAN_SW --- WIN01

    DMZ_SW --- WEB01

    %% =========================================================
    %% FIREWALL INTERNO
    %% =========================================================

    FW_WAN --- FW_ROUTE
    FW_LAN --- FW_ROUTE
    FW_DMZ --- FW_ROUTE
    FW_ROUTE --- NFT
    FW_LAN --- DNSMASQ

    %% =========================================================
    %% DHCP
    %% =========================================================

    CLI01 -.->|"DHCP UDP 67/68"| DNSMASQ
    WIN01 -.->|"DHCP UDP 67/68"| DNSMASQ

    %% =========================================================
    %% DNS
    %% =========================================================

    CLI01 -.->|"DNS UDP/TCP 53"| DNSMASQ
    WIN01 -.->|"DNS UDP/TCP 53"| DNSMASQ

    DNSMASQ -.->|"lab.local → TCP/UDP 53"| SRV01
    DNSMASQ -.->|"DNS externo"| DNS8
    DNSMASQ -.->|"DNS externo"| DNS11

    %% DNS directo dentro de LAN
    CLI01 -.->|"DNS directo TCP/UDP 53<br/>(misma LAN)"| SRV01
    WIN01 -.->|"DNS directo TCP/UDP 53<br/>(misma LAN)"| SRV01

    %% Servidor DNS recursivo
    SRV01 -.->|"Forwarder DNS TCP/UDP 53"| FW_LAN
    FW_LAN -.->|"DNS externo"| DNS8
    FW_LAN -.->|"DNS externo"| DNS11

    %% =========================================================
    %% LAN → SRV01
    %% =========================================================

    CLI01 -.->|"HTTP :80<br/>Intranet"| SRV01
    WIN01 -.->|"HTTP :80<br/>Intranet"| SRV01

    CLI01 -.->|"SMB :445<br/>ventas / desarrollo / publico"| SRV01
    WIN01 -.->|"SMB :445<br/>ventas / desarrollo / publico"| SRV01

    CLI01 -.->|"SSH :22<br/>clave pública"| SRV01
    MON01 -.->|"SSH :22<br/>backup"| SRV01

    %% =========================================================
    %% LAN → DMZ
    %% =========================================================

    CLI01 ==>|"HTTP :80 / HTTPS :443"| FW_DMZ
    WIN01 ==>|"HTTP :80 / HTTPS :443"| FW_DMZ
    MON01 ==>|"TCP :9100<br/>Prometheus scrape"| FW_DMZ

    FW_DMZ ==>|"HTTP :80 / HTTPS :443"| WEB01
    FW_DMZ ==>|"TCP :9100"| WEB01

    CLI01 -.->|"ICMP"| FW_DMZ
    WIN01 -.->|"ICMP"| FW_DMZ

    %% =========================================================
    %% MONITORIZACION
    %% =========================================================

    MON01 ==>|"Prometheus scrape :9100"| FW_LAN
    MON01 ==>|"Prometheus scrape :9100"| SRV01
    MON01 -->|"Prometheus :9090<br/>localhost"| MON01
    MON01 -->|"Node Exporter :9100<br/>localhost"| MON01

    MON01 -->|"Grafana :3000<br/>Acceso UI"| CLI01
    HOSTPC -->|"HTTP :3000"| MON01

    %% =========================================================
    %% BACKUP
    %% =========================================================

    MON01 -.->|"SSH :22<br/>backup / sudo tar"| SRV01
    MON01 -.->|"SSH :22<br/>backup / sudo tar"| WEB01

    MON01 -->|"Restic<br/>/backup/restic"| MON01

    %% Restauracion
    MON01 -.->|"SCP :22<br/>restore"| SRV01
    MON01 -.->|"SCP :22<br/>restore opcional"| WEB01

    %% =========================================================
    %% SEGURIDAD / FAIL2BAN / SSH
    %% =========================================================

    CLI01 -.->|"SSH fallido → fail2ban"| SRV01
    CLI01 -.->|"SSH :22 → fail2ban"| WEB01
    MON01 -.->|"SSH :22"| WEB01

    %% =========================================================
    %% WINDOWS RDP DESDE HOST
    %% =========================================================

    HOSTPC ==>|"RDP :3389<br/>mstsc"| WIN01

    %% El host tiene acceso directo a redes host-only
    HOSTPC -.->|"Acceso directo host-only"| LAN_SW
    HOSTPC -.->|"Acceso directo host-only"| DMZ_SW

    %% =========================================================
    %% SALIDA A INTERNET
    %% =========================================================

    CLI01 ==>|"HTTP :80 / HTTPS :443 / DNS :53"| FW_WAN
    WIN01 ==>|"HTTP :80 / HTTPS :443 / DNS :53"| FW_WAN
    SRV01 ==>|"HTTP :80 / HTTPS :443<br/>actualizaciones"| FW_WAN
    WEB01 ==>|"HTTP :80 / HTTPS :443"| FW_WAN
    MON01 ==>|"HTTP :80 / HTTPS :443<br/>actualizaciones"| FW_WAN

    FW_WAN ==> NAT
    NAT ==> EXT

    %% =========================================================
    %% DMZ → WAN
    %% =========================================================

    WEB01 ==>|"HTTP :80 / HTTPS :443"| FW_WAN

    %% =========================================================
    %% CONEXIONES BLOQUEADAS POR NFTABLES
    %% =========================================================

    WEB01 -.->|"🚫 DMZ → LAN<br/>Bloqueado"| SRV01
    WEB01 -.->|"🚫 DMZ → LAN<br/>SSH :22 bloqueado"| SRV01
    WEB01 -.->|"🚫 DMZ → LAN<br/>SMB :445 bloqueado"| SRV01

    EXT -.->|"🚫 Sin DNAT / Port Forward"| WEB01
    EXT -.->|"🚫 Sin acceso entrante"| SRV01
    EXT -.->|"🚫 Sin acceso entrante"| MON01
    EXT -.->|"🚫 Sin acceso entrante"| FW_LAN

    %% DNS de la DMZ según el tutorial:
    WEB01 -.->|"⚠️ DNS :53 → 10.10.10.1<br/>Configurado pero bloqueado por la política DMZ → LAN"| FW_LAN

    %% =========================================================
    %% ESTILOS
    %% =========================================================

    classDef firewall fill:#ffcccc,stroke:#cc0000,stroke-width:3px
    classDef server fill:#d9ead3,stroke:#38761d,stroke-width:2px
    classDef client fill:#cfe2f3,stroke:#1155cc,stroke-width:2px
    classDef monitor fill:#eadcf8,stroke:#674ea7,stroke-width:2px
    classDef dmz fill:#fce5cd,stroke:#e69138,stroke-width:2px
    classDef wan fill:#eeeeee,stroke:#666666,stroke-width:2px
    classDef switch fill:#fff2cc,stroke:#bf9000,stroke-width:2px

    class FW_WAN,FW_LAN,FW_DMZ,FW_ROUTE,NFT,DNSMASQ firewall
    class SRV01 server
    class MON01 monitor
    class CLI01,WIN01 client
    class WEB01 dmz
    class EXT,DNS8,DNS11,NAT,HOSTPC,VMNET8,VMNET2,VMNET3 wan
    class LAN_SW,DMZ_SW switch
