# 02. Firewall y NAT

---

### 1. **Config interfaces (fw01)**

* **How:**
  * `"sudo nano /etc/network/interfaces"` or go `"ls /etc"` / `"ls /etc/network"` to find out
  * `(Screenshot - Firewall-interfaces_1)`
  * `"systemctl restart networking"`
  * `"sudo nano /etc/sysctl.conf"`
  * `(Screenshot - Firewall-ip-forward_1)`
  * `"sysctl -p"`
  * To be 100% sure about it, force it creating file: `"sudo nano /etc/sysctl.d/99-router.conf"`
  * `(Screenshot - Firewall-ip-forward_2)`

* **Check:**
  * via `"ip a"`, `"cat/proc/sys/net/ipv4/ip_forward"`, `"reboot"` and check again

---

### 2. **Config nftables (fw01)**

* **How:**
  * `"sudo apt update && sudo apt install nftables -y"` and `"sudo nano /etc/nftables.conf"`
  * `(Screenshots - Firewall-nftables_1, Firewall-nftables_2)`

* **Check:**
  * `"nft list ruleset"` -> `"systemctl enable nftables"` -> `"systemctl start nftables"` and `"nft list ruleset > /etc/nftables.conf"` to make it permanent.
  * `(Screenshot - Firewall-connectivity_1)`
