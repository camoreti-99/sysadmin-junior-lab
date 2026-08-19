# 04. Servidores Web (Nginx)

---

### 1. **Create intern web (srv01)**

* **How:**
  * `"sudo apt install nginx -y"` -> `"sudo mkdir -p /var/www/intranet"` -> `"sudo nano /var/www/intranet/index.html"`
  * `(Screenshot - intranet-nginx_1)`
  * `"sudo nano /etc/nginx/sites-available/intranet"`
  * `(Screenshot - intranet-nginx_2)`
  * `**(!!! this config works but in public one not, as the other one [3.] does !!!)**`
  * `"sudo ln -s /etc/nginx/sites-available/intranet /etc/nginx/sites-enabled/"` -> `"sudo rm /etc/nginx/sites-enabled/default"` -> `"sudo nginx -t"`
  * `"sudo systemctl restart nginx"` -> `"sudo systemctl enable nginx"`

* **Check:**
  * `(Screenshot - intranet-nginx_3)`

---

### 2. **Check connectivity DMZ (web01)**

* **How:**
  * `(Screenshots - DMZ_1, DMZ_2)`

---

### 3. **Create extern web (web01)**

* **How:**
  * `"sudo apt install nginx -y"` -> `"sudo mkdir -p /var/www/web01"` -> `"sudo nano /var/www/web01/index.html"`
  * `(Screenshot - extern-nginx_1, extern-nginx_2)`
  * `"sudo nano /etc/ngninx/sites-available/web01"`
  * `(Screenshot - extern-nginx_3)`
  * `**(!!! this config works but in intern one not, as another one [1.] does !!!)**`
  * `"sudo ln -s /etc/nginx/sites-available/web01 /etc/nginx/sites-enabled/"`
  * `"sudo rm /etc/nginx/sites-enabled/default`
  * `"sudo nginx -t"`
  * `"sudo systemctl restart nginx"`
  * `"sudo systemctl enable nginx"`

* **Check:**
  * `"curl http://127.0.0.1"`
  * `(Screenshot - extern-nginx_4)`
