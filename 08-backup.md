# 08. Estrategia de Copias de Seguridad (Restic)

---

### 1. **Config restic (mon01):**

* **How:**
  * `"sudo apt install restic -y"`
  * Add disk (optional):
  * `"sudo fdisk /dev/sdb"   # create partition: n, p, 1, enter, enter, w`
  * `"sudo mkfs.ext4 /dev/sdb1"`
  * `"sudo mkdir /backup"`
  * `"sudo mount /dev/sdb1 /backup"`
  * Make it permanent:
  * `(Screenshot - backup-restic_1)`
  * `"sudo mount -a"`
  * `export RESTIC_PASSWORD="p@ssw0rd"`
  * `restic init --repo /backup/restic`

* **Check:**
  * No errors in previous steps, everything OK

---

### 2. **Backup script (mon01):**

* **How:**
  * `(Screenshot - backup-script_1)`
  * `"sudo visudo" in srv01 and web01`
  * `(Screenshot - backup-script_2)`
  * `"sudo chmod +x /usr/local/bin/backup_restic.sh"`
  * `(Manual exec: "sudo /usr/local/bin/backup_restic.sh")`
  * `!!! DO THIS AFTER CHECK !!!`
  * `sudo crontab -e`
  * `(Screenshot - backup-script_4)`

* **Check:**
  * `(Screenshot - backup-script_3)`

---

### 3. **File restoring plan (srv01 & mon01):**

* **How:**
  * `(Screenshot - backup-test_1)`
  * `"sudo /usr/local/bin/backup_restic.sh"`
  * `(Screenshot - backup-test_2, backup-test_3)`
  * `If u did a “clear” and can’t see snapshot ID, hit: "sudo RESTIC_REPOSITORY="/backup/restic" RESTIC_PASSWORD="p@ssw0rd" restic snapshots"`
  * `(Screenshot - backup-test_4)`
  * `"sudo RESTIC_REPOSITORY="/backup/restic" RESTIC_PASSWORD="p@ssw0rd" restic restore id1234 --target /tmp/restore"`
  * `"scp /tmp/restore/srv01_compartido.tar sysadmin@10.10.10.10:/tmp/`   //   `scp /tmp/restore/web01_www.tar sysadmin@10.10.20.10:/tmp/"`
  * `(Screenshot - backup-test_5)`
  * In OG path:
  * `!!! Possible troubleshooting: when coming back to srv, hit “cd /srv” and not “cd/srv/compartido”, otherwise the backup will be restored in “cd/srv/compartido/compartido”. !!!`
  * The backup already generates whole “compartido” once inside “/srv”.
  * `"cd /srv && sudo tar -xf /tmp/srv01_compartido.tar" or "cd /var/www && sudo tar -xf /tmp/web01_www.tar"`

* **Check:**
  * `"sudo ls -l /srv/compartido/ventas/importante.txt" or "sudo ls -l /var/www/web01/index.html"`
