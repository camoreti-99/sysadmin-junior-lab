# 05. Servidor de Ficheros Samba

1. **Users and groups (srv01):**
    - **How**:
        "sudo groupadd ventas" -> "sudo groupadd desarrollo" -> "sudo groupadd admin_lab"

        "sudo useradd -m -s /bin/bash -G ventas -p $(openssl passwd p@ssw0rd) carlos" 
        "sudo useradd -m -s /bin/bash -G desarrollo -p $(openssl passwd p@ssw0rd) ana"
        "sudo useradd -m -s /bin/bash -G admin_lab -p $(openssl passwd p@ssw0rd) sysadmin"

        "sudo chage -d 0 carlos"
        "sudo chage -d 0 ana"
        "sudo chage -d 0 sysadmin"
    
    - **Check:**
        (Screenshot - Create-users-and-groups_1)

2. **Config Samba (srv01):**
    - **How:**
        sudo apt install samba samba-common smbclient -y
        sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

        "sudo mkdir -p /srv/compartido/ventas"
        "sudo mkdir -p /srv/compartido/desarrollo"
        "sudo mkdir -p /srv/compartido/publico"

        "sudo chgrp ventas /srv/compartido/ventas"
        "sudo chgrp desarrollo /srv/compartido/desarrollo"

        "sudo chmod 770 /srv/compartido/ventas"
        "sudo chmod 770 /srv/compartido/desarrollo"
        "sudo chmod 755 /srv/compartido/publico"
        
        "sudo chmod 1777 /srv/compartido/publico" - Stickybit (no restrictions and nobody but who created the file can remove it)

        "sudo nano /etc/samba/smb.conf"
        (Screenshot - Samba_1)

    - **Check:**
        (Screenshot - Samba_2)

3. **Create Samba Users (srv01):**
    - **How:**
        sudo smbpasswd -a carlos
        sudo smbpasswd -a ana
        sudo smbpasswd -a sysadmin
        (Password will be asked: p@ssw0rd)

        sudo systemctl restart smbd nmbd
        sudo systemctl enable smbd nmbd
    
    -  **Check:**
        
        Before, login and password will be asked to change, so u will be able check as well:

        sudo smbpasswd carlos
        sudo smbpasswd ana
        sudo smbpasswd sysadmin

        (Screenshot - Samba_3)




4. **Mount Samba (cli01)**
    - **How and check:**
        smb://10.10.10.10/ventas
        (Screenshots - Client_13, Client_14, Client_15)
        Permanent:
        (Screenshots - Client_16, Client_17, Client_18)
