### 1. **Final checks:**

* **How:**
  * `(Screenshots - Final-checks_1, Final-checks_2, Final-checks_3, Final-checks_4, Final-checks_5, Final-checks_6, Final-checks_7, Final-checks_8, Final-checks_9, Final-checks_10, Final-checks_11, Final-checks_12, Final-checks_13)`

---

# Incident Resolution & Troubleshooting Command Reference

---

## 1. Network & Firewall Diagnostics (`nftables` / `dnsmasq`)

### Common Incidents (Loss of connectivity, DHCP assignment)

* **`ip -br a`**
  Displays all system network interfaces with their IP addresses and operational statuses (UP/DOWN) in a clean tabular format.

* **`ip route show`**
  Shows the kernel routing table to verify default gateways and outgoing interface paths.

* **`ping -c 3 10.10.10.1`**
  Sends 3 ICMP Echo packets to the specified IP to verify basic network layer connectivity.

* **`ss -tulnp | grep -E "dnsmasq|sshd"`**
  Displays open listening TCP/UDP sockets alongside their associated process names and PIDs.

### Advanced Incidents (NAT failure, firewall block, ARP collisions)

* **`nft list ruleset`**
  Displays all active filtering and NAT tables, chains, and rules loaded in `nftables`.

* **`nft -c -f /etc/nftables.conf && nft -f /etc/nftables.conf`**
  Validates the syntax of the firewall configuration file first (`-c`), and applies it (`-f`) only if no errors are detected.

* **`ip neighbor flush all`**
  Flushes the local ARP cache, forcing the system to re-resolve MAC addresses across the network.

* **`tcpdump -i ens34 -n icmp or port 53`**
  Captures real-time network traffic on interface `ens34`, filtering specifically for ICMP (ping) or DNS (port 53) without resolving IP hostnames (`-n`).

* **`cat /proc/sys/net/ipv4/ip_forward`**
  Displays IP packet forwarding status in the kernel; a value of `1` indicates the machine is routing traffic.

---

## 2. DNS Name Resolution (`BIND9` / `dnsmasq`)

### Common Incidents (Incorrect records, resolution failures)

* **`dig @127.0.0.1 srv01.lab.local`**
  Queries the local DNS server (`127.0.0.1`) directly to resolve the A record (IP) for the specified domain.

* **`dig @127.0.0.1 -x 10.10.10.10`**
  Performs a reverse DNS lookup (PTR query) to retrieve the hostname associated with the IP address.

### Advanced Incidents (Zone syntax errors, delegation issues)

* **`named-checkconf /etc/bind/named.conf.local`**
  Validates the syntax of BIND9 configuration files before restarting the service to prevent downtime.

* **`named-checkzone lab.local /etc/bind/zones/db.lab.local`**
  Validates the syntax, serial number, and resource record structure of a forward lookup zone file.

* **`named-checkzone 10.10.10.in-addr.arpa /etc/bind/zones/db.10.10.10`**
  Validates the syntax and PTR records of a reverse lookup zone file.

* **`dig +trace +nodnssec lab.local`**
  Traces the full iterative DNS resolution path from root servers down to the authoritative answer, omitting DNSSEC validation to isolate delegation faults.

* **`rndc reload`**
  Reloads BIND9 global configurations and zone files gracefully without restarting the daemon or dropping active connections.

---

## 3. Web Servers & Proxies (`Nginx`)

### Common Incidents (Service down, 50x / 404 errors)

* **`systemctl status nginx`**
  Displays current Nginx service status (active, inactive, failed) alongside recent system log entries.

* **`curl -I [http://127.0.0.1](http://127.0.0.1)`**
  Sends an HTTP request retrieving only response headers (`-I`) to quickly inspect status codes (200 OK, 403, 404, 500).

### Advanced Incidents (Port conflicts, hidden syntax errors)

* **`nginx -t`**
  Tests all Nginx configuration files for syntax errors and missing file paths.

* **`nginx -T`**
  Validates syntax and dumps the entire active in-memory Nginx configuration to stdout.

* **`lsof -i :80 -i :443`**
  Identifies processes currently binding or blocking standard HTTP (80) and HTTPS (443) web ports.

* **`tail -f /var/log/nginx/error.log`**
  Streams the last lines of the Nginx error log, updating in real time as events occur.

---

## 4. Storage, LVM & File Sharing (`Samba` / `CIFS` / `Restic`)

### Common Incidents (SMB auth failure, unmounted shares)

* **`testparm`**
  Validates the syntax of `/etc/samba/smb.conf` and prints active shared resources.

* **`smbclient -L //127.0.0.1 -U carlos`**
  Lists available shared folders on the local Samba server authenticating as the specified user.

* **`mount -t cifs //srv01/ventas /mnt/ventas -o username=carlos`**
  Mounts a remote Samba share onto a local directory using CIFS.

### Advanced Incidents (SMB file locks, full partitions, locked Restic repository)

* **`smbstatus`**
  Displays active Samba connections, mounted shares, and currently locked/opened files in real time.

* **`lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv -r`**
  Expands an LVM logical volume to consume all remaining free space and automatically resizes the underlying filesystem (`-r`).

* **`restic unlock`**
  Removes stale lock files from a Restic repository left behind after interrupted backup operations.

* **`restic check`**
  Verifies structural integrity and data blob consistency within the backup repository.

* **`lsof +L1`**
  Identifies unlinked (deleted) files that remain held open by running processes, preventing disk space reclamation.

---

## 5. Security, Access Control & SSH (`Fail2ban` / Keys)

### Common Incidents (Banned IP, SSH auth failure)

* **`fail2ban-client status sshd`**
  Displays SSH jail status in Fail2ban, including failed attempt counters and currently banned IPs.

* **`fail2ban-client set sshd unbanip 10.10.10.100`**
  Instantly unbans a specified IP address, restoring its access to SSH.

### Advanced Incidents (SSH handshake debugging, key permissions)

* **`ssh -vvv sysadmin@10.10.10.10`**
  Runs an SSH client connection in verbose debug mode (`-vvv`) to pinpoint authentication or handshake failures.

* **`ssh -o PubkeyAuthentication=no sysadmin@10.10.10.10`**
  Disables public key authentication to verify whether password fallback is accepted or blocked.

* **`chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`**
  Enforces strict permission standards required for the SSH directory and keys to prevent `sshd` from rejecting them.

* **`journalctl -u ssh -n 50 --no-pager`**
  Displays the last 50 SSH daemon log entries directly from the `systemd` journal.

---

## 6. System Performance & Diagnostics (CPU, Memory, I/O, Processes)

### Common Incidents (High CPU/RAM usage)

* **`htop`**
  Interactive process viewer providing real-time CPU, RAM, and Swap utilization metrics.

* **`df -h`**
  Displays disk space usage, total capacity, and available space for all mounted filesystems in human-readable format.

* **`free -h`**
  Reports total, used, free, and cached system RAM alongside Swap utilization.

### Advanced Incidents (Zombie processes, high I/O wait, FD exhaustion)

* **`ps aux | awk '$8 ~ /Z/ { print $0 }'`**
  Filters all running processes to isolate "Zombie" (defunct) processes awaiting parent process cleanup.

* **`iotop -oP`**
  Filters and displays only processes or threads actively performing disk read/write I/O operations.

* **`cat /proc/sys/fs/file-nr`**
  Displays allocated vs. unused open file descriptors against the kernel maximum limit.

* **`systemctl restart nginx || journalctl -u nginx -e --no-pager`**
  Attempts to restart Nginx; if it fails, immediately dumps the end of the service log without pagination.

---

## 7. Monitoring & Metrics (`Prometheus` / `Node Exporter`)

### Common Incidents (Metrics unavailable, endpoint down)

* **`curl http://localhost:9100/metrics`**
  Queries Node Exporter locally to check if system metrics are being generated and exposed.

* **`curl http://localhost:9090/targets`**
  Queries the Prometheus API to check target scrape statuses (verifying if targets are "UP").

### Advanced Incidents (Prometheus config validation)

* **`promtool check config /etc/prometheus/prometheus.yml`**
  Validates the YAML syntax and configuration structure of Prometheus before reloading.
