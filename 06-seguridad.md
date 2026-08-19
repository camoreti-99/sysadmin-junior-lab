# 06. Hardening y Seguridad

1. **Config fail2ban (srv01)**
    - **How:**
        "sudo apt install fail2ban -y" -> "sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local" -> "sudo nano /etc/fail2ban/jail.local"
        (Screenshot - Fail2ban_1)

        "sudo systemctl restart fail2ban" -> "sudo systemctl enable fail2ban"
    
    - **Check:**
        (Screenshot - Fail2ban_2)

2. **Config fail2ban (web01**)
    - **Follow same steps as in [1.]**

3. **Config ssh to srv01 and web01 (cli01):**
    - **How:**
        (Screenshot - Client_9, Client_10)
        (After copy ssh id, turn off passwordauthentication and then try to connect)
    
    - **Check:**
        (Screenshot - Client_11, Client_12)




4. **Config ssh to srv01 and web01 (mon01):**
    - **How:**
        (After copy ssh id, turn off passwordauthentication and then try to connect)
        ssh-keygen -t ed25519 -C "backup@mon01" -> (No passphrase [or yes]) -> "ssh-copy-id sysadmin@10.10.10.10" -> "ssh-copy-id sysadmin@10.10.20.10"
    - **Check:**
        (Screenshot - Monitoring-SSH_1)