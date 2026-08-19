## 03. DNS y DHCP

---

### 1. **Config dnsmasq (fw01)**

* **How:**
  * `"sudo apt install dnsmasq -y"` -> `"sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak"` -> `"sudo sh -c 'echo "" > /etc/dnsmasq.conf'"` -> `"sudo nano /etc/dnsmasq.conf"`
  * `(Screenshot - dnsmasq_1)`
  * `"systemctl restart dnsmasq"` -> `"systemctl enable dnsmasq"` -> `"systemctl status dnsmasq"`

* **Check:**
  * `"sudo ss -ulnp | grep dnsmasq"`
  * `(Screenshot - dnsmasq_2, dnsmasq_3)`

---

### 2. **Check connectivity (srv01)**

* **How:**
  * `(Screenshot - Server-connectivity_1)`

---

### 3. **Config bind9 - authoritative DNS (srv01)**

* **How:**
  * `"sudo apt install bind9 bind9utils bind9-doc dnsutils -y"`
  * `"sudo nano /etc/bind/named.conf.options"`
  * `(Screenshot - bind9_1)`
  * `"sudo nano /etc/bind/named.conf.local"`
  * `(Screenshot - bind9_2)`
  * `"sudo nano /etc/bind/zones/db.lab.local"`
  * `(Screenshot - bind9_3)`
  * `"sudo nano /etc/bind/zones/db.10.10.10"`
  * `(Screenshot - bind9_4)`

* **Check:**
  * Everything must return OK or nothing:
  * `"sudo named-checkconf /etc/bind/named.conf.local"`
  * `"sudo named-checkzone lab.local /etc/bind/zones/db.lab.local"`
  * `"sudo named-checkzone 10.10.10.in-addr.arpa /etc/bind/zones/db.10.10.10"`
  * `"sudo systemctl restart bind9"` -> `"sudo systemctl enable bind9"` -> `"sudo systemctl status bind9"`
  * Everything must say status: NOERROR
  * `dig @127.0.0.1 srv01.lab.local`
  * `dig @127.0.0.1 web01.lab.local`
  * `dig @127.0.0.1 -x 10.10.10.10`

---

### 4. **Check Ubuntu Desktop client DHCP & services (cli01)**

* **How:**
  * `(Screenshots - Client_1, Client_2)`
  * Possible trouble (force DNS) -> `(Screenshot - Client_3)`
  * `(Screenshots, Client_4, Client_5, Client_6, Client_7, Client_8)`
  * `"sudo apt update"` -> `"sudo apt install nmap dnsutils smbclient curl htop traceroute -y"`

---

### 5. **Check Windows Pro client DHCP & Services (win01)**

* **How:**
  * `(Screenshots - Client-Windows_1, Client-Windows_2, Client-Windows_3, Client-Windows_4)`
  * This PC -> Connect to net unit
  * `(Screenshots - Client-Windows_5, Client-Windows_6)`
  * Possible trouble (edit gpedit.msc to enable guest unsafe login):
  * `(Screenshots - Client-Windows_7)`
