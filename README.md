# Laboratorio de Infraestructura y Servicios | Carlos Amoretti

Este repositorio contiene la documentación técnica, configuraciones y scripts desplegados en el laboratorio multiplataforma del ciclo ASIR.

## Resumen Ejecutivo
La infraestructura está compuesta por 6 máquinas virtuales sobre VMware Workstation Pro 17+, organizadas en tres zonas de red: WAN, LAN y DMZ.
- **fw01:** Firewall y router principal (Debian 12) con enrutamiento, nftables, NAT y DHCP/DNS forwarding con dnsmasq.
- **srv01:** Servidor de infraestructura en LAN (Ubuntu Server 24.04) con DNS autoritativo (bind9), Nginx Intranet y Samba.
- **web01:** Servidor web en DMZ (Ubuntu Server 24.04) para servicios públicos.
- **mon01:** Centro de monitorización y copias de seguridad (Ubuntu Server 24.04) con Prometheus, Grafana y restic.
- **cli01 / win01:** Clientes de prueba en LAN (Ubuntu Desktop 24.04 y Windows 11 Pro).


## Estructura del Repositorio
- `01-arquitectura.md`
- `02-firewall.md`
- `03-dns-dhcp.md`
- `04-servidor-web.md`:
- `05-ficheros-permisos.md`
- `06-seguridad.md`
- `07-monitorizacion.md`
- `08-backup.md`
- `09-automatizacion.md`
- `10-anexos.md`

- Paso a paso detallado siguiendo el principio de:
    **Que quiero**
    **Como lo hago**   
    **Evidencia de que funciona**


