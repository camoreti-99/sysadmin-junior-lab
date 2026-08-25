# Active Directory Client (win02)

---

### 1. **Config network and rename VM**

* **How:**
  * `Same that [1.] and [2.] in 11-Active-Directory`

* **Check:**
  * `(Screenshots - windows-client-domain_1, windows-client-domain_2)`

---

### 2. **Check DNS before merge**

* **How:**
  * `"Resolve-DnsName dc01.amoretti.local"`
  * `"Resolve-DnsName amoretti.local"`

  * `"Test-NetConnection dc01.amoretti.local -Port 53"`
  * `"Test-NetConnection dc01.amoretti.local -Port 88"`
  * `"Test-NetConnection dc01.amoretti.local -Port 445"`

  * `(Screenshot - windows-client-domain_3)`

  ---

### 3. **Merge win02 to the domain**

* **How:**
  * `"Add-Computer ` -DomainName "amoretti.local" ` -Credential (Get-Credential) ` -Restart"`

  * `(Screenshot - windows-client-domain_4)`

* **Check:**
  * `(Screenshots - windows-client-domain_5, windows-client-domain_6, windows-client-domain_7, windows-client-domain_8)`

  * ***EXTRA: Remove OG user from vm w11pro: `(Screenshot - windows-client-domain_9)`***

  ---


### 4. **Move win02 to the right OU (dc01)**

* **How:**
  * `"Get-ADComputer -Identity "win02""`
  * `"Move-ADObject ` -Identity (Get-ADComputer "win02").DistinguishedName ` -TargetPath "OU=Windows,OU=Equipos,OU=EMPRESA,DC=amoretti,DC=local""`

* **Check:**
  * `"Get-ADComputer -Identity "win02" | Select Name,DistinguishedName"`

  * `(Screenshot - windows-client-domain_10)`

  ---


### 5. **Create GPO and set a policy as example (dc01)**

* **How:**
  * `"New-GPO -Name "GPO-Workstations-Baseline""`, `"Get-GPO -All | Select DisplayName"`
  * `"New-GPLink ` -Name "GPO-Workstations-Baseline" ` -Target "OU=Windows,OU=Equipos,OU=EMPRESA,DC=amoretti,DC=local""`
  * `"Set-GPRegistryValue ` -Name "GPO-Workstations-Baseline" ` -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\System" ` -ValueName "VerboseStatus" ` -Type DWord ` -Value 1"`
  

* **Check (win02):**
  * `"gpupdate /force"`, `"gpresult /r"`

  * `(Screenshots - windows-client-domain_11, windows-client-domain_12, windos-client-domain_13)`

  ---

### 6. **Create file server and its resource (dc01)**

* **How:**
  * `"New-Item -Path "C:\Empresa" -ItemType Directory"`
  * `"New-Item -Path "C:\Empresa\Ventas" -ItemType Directory"`
  * `"New-Item -Path "C:\Empresa\Desarrollo" -ItemType Directory"`
  * `"New-Item -Path "C:\Empresa\Publico" -ItemType Directory"`

  * `"New-SmbShare ` -Name "Ventas" ` -Path "C:\Empresa\Ventas" ` -ChangeAccess "AMORETTI\GG_Ventas""`
  * `"New-SmbShare ` -Name "Desarrollo" ` -Path "C:\Empresa\Desarrollo" ` -ChangeAccess "AMORETTI\GG_Desarrollo""`
  * `"Get-SmbShare"`

  * `(Screenshots - windows-client-domain_14, windows-client-domain_15)`
  

* **Check:**
  * Win02 - `"Test-Path "\\dc01\Ventas""`, `"Test-Path "\\dc01\Desarrollo""`

  * Win02 - `"New-Item "\\dc01\Ventas\prueba.txt" -ItemType File"`, `"Get-ChildItem "\\dc01\Ventas""`
  * dc01 - `"Get-ChildItem "\\dc01\Ventas""`

  * `(Screenshots - windows-client-domain_16, windows-client-domain_17, windos-client-domain_18)`

