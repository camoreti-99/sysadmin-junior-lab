# Active Directory (dc01)

---

### 1. **Check and rename windows server**

* **How:**
  * `"Rename-Computer -NewName "dc01" -Force"`, `"Restart-Computer"`

* **Check:**
  * `"hostname"` -> dc01
  
  * `(Screenshots - windows-server_1, windows_server_2)`

---

### 2. **Config network**

* **How:**
  * `"Get-NetIPConfiguration"` to see "Ethernet0" or whatever interface name
  * `"Get-NetIPAddress -InterfaceAlias "Ethernet0" -AddressFamily IPv4"`

  * `"New-NetIPAddress `-InterfaceAlias "Ethernet0"` -IPAddress 10.10.10.30 `-PrefixLength 24` -DefaultGateway 10.10.10.1"`
  * `"Set-DnsClientServerAddress ` -InterfaceAlias "Ethernet0" ` -ServerAddresses 10.10.10.30"`

  * `(Screenshot - windows-server_3)`

* **Check:**
  * `"Get-NetIPConfiguration"`
  * `(Screenshot - windows-server_4)`

---

### 3. **check connectivity**

* **How:**
  * `"ping 10.10.10.1"`
  * `"ping 10.10.10.10"`
  * `"ping 10.10.10.20"`

  * `"Test-NetConnection 8.8.8.8"` and `"Test-NetConnection google.com -Port 443"` (must fail but Ok connect)

  * `(Screenshots - windows-server_7, windows_server_8)`
  * ***Screenshots 5 and 6 just for make net private and check arp to fw if connectivity doesnt work***

---

### 4. **Install AD domain**

* **How:**
  * `"Install-WindowsFeature AD-Domain-Services -IncludeManagementTools"`

* **Check:**
  * `"Get-WindowsFeature AD-Domain-Services"` must show `[X] Active Directory Domain Services`

  * `(Screenshot - windows-server_9)`

---

### 5. **Create AD forest**

* **How:**
  * `"$DSRM = Read-Host "Introduce contraseña DSRM" -AsSecureString"`
  * `"Install-ADDSForest ` -DomainName "amoretti.local" ` -InstallDNS ` -SafeModeAdministratorPassword $DSRM ` -Force "`

  * `(Screenshot - windows-server_9)`

* **Check:**
  * log in as `"AMORETTI\Administrator"` and `"whoami"`

  * `(Screenshots - windows-server_10, windows-server_11)`

---

### 6. **Check AD**

* **How:**
  * `"Get-ADDomain"`
  * `"Get-ADForest"`
  * `"Get-ADDomainController"`
  * `"Get-Service DNS"`
  * `"Resolve-DnsName dc01.amoretti.local"`
  * `"Resolve-DnsName amoretti.local"`

  * `(Screenshots - windows-server_12, windows-server_13, windows-server_14, windows-server_15)`

  ---

### 7. **Check SYSVOL & NETLOGON**

* **How:**
  * `"Get-SmbShare"`, `"Get-Service NTDS,DNS,Netlogon"`

  * `(Screenshot - windows-server_16)`

  * `"dcdiag"` for diagnostics

  * `(Screenshot - windows-server_17)`

    ---

### 8. **Create company estructure (ALL CLI no clicks)**

* **How:**
  * `"New-ADOrganizationalUnit -Name "EMPRESA" -Path "DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Usuarios" ` -Path "OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Equipos" ` -Path "OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Grupos" ` -Path "OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Ventas" ` -Path "OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Desarrollo" ` -Path "OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "IT" ` -Path "OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADOrganizationalUnit ` -Name "Windows" ` -Path "OU=Equipos,OU=EMPRESA,DC=amoretti,DC=local""`

* **Check:**
  * `"Get-ADOrganizationalUnit -Filter * | Select-Object Name,DistinguishedName"`

  * `(Screenshot - windows-server_18)`

      ---

### 9. **Create groups**

* **How:**
  * `"New-ADGroup ` -Name "GG_Ventas" ` -GroupScope Global ` -GroupCategory Security ` -Path "OU=Grupos,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADGroup ` -Name "GG_Desarrollo" ` -GroupScope Global ` -GroupCategory Security ` -Path "OU=Grupos,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADGroup ` -Name "GG_IT" ` -GroupScope Global ` -GroupCategory Security ` -Path "OU=Grupos,OU=EMPRESA,DC=amoretti,DC=local""`

* **Check:**
  * `"Get-ADGroup -Filter * | Select Name,GroupScope,GroupCategory"`

  * `(Screenshots - windows-server_19, windows-server_20)`

     ---

### 10. **Create Users and add 'em to groups**

* **How:**
  * `"$Password = Read-Host "Temp passwrd" -AsSecureString"` -> Password required with CAPS symbols and numbers, othetwise when create user it won't work
  * `"New-ADUser ` -Name "Carlos Amoretti" ` -SamAccountName "carlos.amoretti" ` -UserPrincipalName "carlos.amoretti@amoretti.local" ` -AccountPassword $Password ` -Enabled $true ` -Path "OU=Ventas,OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADUser ` -Name "Ana Garcia" ` -SamAccountName "ana.garcia" ` -UserPrincipalName "ana.garcia@amoretti.local" ` -AccountPassword $Password ` -Enabled $true ` -Path "OU=Desarrollo,OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"New-ADUser ` -Name "Ana Garcia" ` -SamAccountName "it.admin" ` -UserPrincipalName "it.admin@amoretti.local" ` -AccountPassword $Password ` -Enabled $true ` -Path "OU=IT,OU=Usuarios,OU=EMPRESA,DC=amoretti,DC=local""`

  * `"Add-ADGroupMember -Identity "GG_Ventas" -Members "carlos.amoretti""`
  * `"Add-ADGroupMember -Identity "GG_Desarrollo" -Members "ana.garcia""`
  * `"Add-ADGroupMember -Identity "GG_IT" -Members "it.admin""`

  * `(Screenshot - windows-server_21)`
  

* **Check:**
  * `"Get-ADGroupMember GG_Ventas"`, `"Get-ADGroupMember GG_Desarrollo"`, `"Get-ADGroupMember GG_IT"`

  * `(Screenshot - windows-server_22)`

      ---

### 11. **Add DNS forwarder (lab.local) - [do 12-Active-Directory-Client (win02) first, recommended]**

* **How:**
  * `"Get-Service DNS"`
  * `"Add-DnsServerConditionalForwarderZone ` -Name "lab.local" ` -MasterServers 10.10.10.10 ` -PassThru"`, `"Get-DnsServerZone"`
  * `(Screenshot - Active-Directory-DNS-add_1)`

* **Check:**
  * `"Test-NetConnection 10.10.10.10 -Port 53"`, `"Resolve-DnsName srv01.lab.local -Server 10.10.10.10"`, `"Resolve-DnsName web01.lab.local -Server 10.10.10.10"`
  * `"Get-DnsServerZone -Name "lab.local""`

  * `(Screenshots - Active-Directory-DNS-add_2, Active-Directory-DNS-add_3)`

  * Win02 - `"Test-NetConnection 10.10.10.10 -Port 80"`, `"Test-NetConnection 10.10.20.10 -Port 80"`, `"Test-NetConnection srv01.lab.local -Port 80"`, `"Test-NetConnection web01.lab.local -Port 80"`
  * Win02 - `"nslookup srv01.lab.local"`, `"nslookup web01.lab.local"`

  * `(Screenshots - Active-Directory-DNS-add_4, Active-Directory-DNS-add_5, Active-Directory-DNS-add_6, Active-Directory-DNS-add_7)`
