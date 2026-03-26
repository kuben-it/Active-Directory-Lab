# Active Directory-lab
Her finner du instruksjoner og eksempel-dokumentasjon for et Windows-basert nettverk av virtuelle maskiner ved hjelp av Active Directory Domain Services (AD DS). 

## Status
1. Oppsett - DONE✅
2. Domenekontroller (Windows Server) - DONE✅
3. DHCP - DONE ✅
4. Klienter - WIP 🏗️
5. Fildeling - coming soon... ❌
6. GPO - coming soon... ❌

## Arkitektur:
### Maskin 1. Domain Controller (DC1)
* OS: Windows Server
* Tjenester/roller: Active Directory, DNS, DHCP
* IP: 192.168.10.10

### Maskin 2. Klient (CL1)
* OS: Windows 10
* Tjenester/roller: DHCP-klienter, domene-medlemmer
* IP range: 192.168.10.100-200

### Virtuelt nettverk
* 1 virtuell switch
* DHCP fra DC
* Default gateway: 192.168.10.1

### Mulige utvidelser
* VLAN, segmentering
* Cross-platform med Linux-klienter
* NAT-router for internett-tilgang
* Scripting, automatisering, backups, logging...
* WDS eller tilsvarende for oppsett av nye klienter
* Sysinternals/monitoring tools
* Back-up domenekontroller

## Læringsmål
Dette er ting du kan bruke og forklare når du har satt opp ditt eget system er ferdig:
* AD, AD DS, DC
* DNS, DHCP
* Group Policy, Group Policy Object, Organizational Unit
* Subnet

