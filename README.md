# Bedriftsnettverk
Her skal jeg lære meg å sette opp et Windows-basert nettverk i virtuelle maskiner ved hjelp av Hyper-V. 

## Status
1. Oppsett - DONE✅
2. Domenekontroller (Windows Server) - DONE✅
3. DHCP - DONE ✅
4. Klienter - WIP 🏗️
5. Fildeling - coming soon... ❌
6. GPO - coming soon... ❌
7. Kanskje: segmentering av nettverk, AD i Azure (Entra ID)

## Tenkt arkitektur:
### 1. Domain Controller (DC1)
* OS: Windows Server
* Tjenester/roller: Active Directory, DNS, DHCP
* IP: 192.168.10.10

### 2. File Server (FS1)
* OS: Windows Server
* Tjenester/roller: Fildeling (SMB), backup og logging, Event Viewer?
* IP: 192.168.10.20

### 3. Tre klienter (CL1, CL2, CL3)
* OS: Windows 11 og Windows 10
* Tjenester/roller: DHCP-klienter, domene-medlemmer,
* Annet: en IT-administrator og to andre ansatte i ulike avdelinger
* IP range: 192.168.10.100-200

### 4. Virtuelt nettverk
* 1 virtuell switch
* DHCP fra DC
* Default gateway: 192.168.10.1

### 5. Mulige utvidelser
* VLAN, segmentering
* Cross-platform med Linux-klienter
* Flere tjenester: intern chat, wiki, mer ?
* "NAT-router for internett-tilgang"?
* Scripting, automatisering, backups, logging... Av hva da?
* WDS eller tilsvarende for oppsett av nye klienter (tror jeg)
* Sysinternals/monitoring tools
* (Research) back-up domenekontroller

## Læringsmål
Dette er ting jeg vil kunne bruke og forklare opp når jeg er ferdig:
* AD, AD DS, DC
* DNS, DHCP
* SMB og tilgangsrettigheter
* GPO og klientstyring
* Subnet, maske, 
* PowerShell 
* Nettverkstegning og -dokumentasjon

## Løse notater
Når vi setter opp en DHCP-server, velger vi selv en IP-range som deles ut til klientene. Eks.: 192.168.10.100 til 192.168.10.200 gir 100 adresser til klienter.
Vi gir gjerne tjenestene statisk IP utenfor denne rangen. F eks. gir jeg her DC1 statisk IP 192.168.10.10. Hvis ikke kan vi risikere at DHCP-serveren gir ut en IP som allerede er i bruk av en tjeneste utenfor DHCP-en.
Default gateway er routerens ***lokale IP-adresse*** på nettverket. Det vil si at det er hit enhetene på nettverket sender data som skal ut av det lokale nettverket. Lett å forveksle med offentlig IP, som er adressen eksterne enheter sender data til for å nå ditt nettverk.

![image](https://github.com/user-attachments/assets/634f928b-093b-4f2b-8bba-564c9f01f792)

Burde se om jeg kan finne script for å installere et stooort AD DS system? Eller finne en slags "case" eller noe.

---
## Hvordan DHCP fungerer
// Vi setter opp DHCP selv isf å la routeren håndtere det (blant annet) fordi den da kan integreres med AD!

🔧 Når du har satt opp DHCP-serveren:

    Ja – nye klienter vil automatisk få en IP-adresse fra DHCP-serveren, så lenge de er koblet til samme virtuelle switch og DHCP er riktig konfigurert og aktivert.

🔁 Hva skjer da, steg for steg?

    Klienten starter og sier:

DHCP Discover → "Hei! Er det noen DHCP-server der ute?"

DHCP-serveren svarer med:

DHCP Offer → "Ja! Her er IP 192.168.10.103, vær så god!"

Klienten sier:

DHCP Request → "Takk! Jeg vil gjerne bruke den."

DHCP-serveren sier:

    DHCP Ack → "IP-en er din i 24 timer."

Denne prosessen skjer helt automatisk, så lenge:

    Klienten er satt til "obtain IP automatically"

    DHCP-serveren er i samme nettverk (samme virtuelle switch, samme subnet)

    DHCP-serveren kjører og har aktive scopes

🕳️ Men hva skjer før du har satt opp DHCP?

Uten en DHCP-server:

    Klienten prøver samme prosess – men ingen svarer

    Etter en stund gir klienten seg og tildeler seg selv en APIPA-adresse

🟡 APIPA = Automatic Private IP Addressing

    Typisk i området 169.254.x.x

    Brukes bare for midlertidig lokal kommunikasjon

    Kan ikke kommunisere med andre enheter i nettverket (annet enn tilfeldigvis andre APIPA-klienter)

    APIPA = en slags "nødløsning" fra Windows når det ikke finnes DHCP


## NAT (Network Address Translation)
Når en klient sender en HTTP(S) ut av nettverket via default gateway, bytter routeren "avsenderaddressen" fra den lokale IPen med sin egen offentlige IP når trafikken forlater nettverket. Forespørselen sendes ut av en bestemt ekstern port, og samtidig lagrer i en intern tabell hvilken lokale addresse svar på denne porten skal vidersendes til. Så hvis routeren velger å sende trafikk fra klient A via sin port 4000, vet den at når svaret kommer tilbake til port 4000 skal dette sendes til klient A.

Den interne tabellen med oversikt over aktiv kommunikasjon(?) kalles NAT-tabellen. Radene her slettes når forbindelsen avsluttes eller etter inaktivitet.

Internettjenester kan ikke vite hvem klienten er gjennom TCP/NAT alene - det kan jo være flere brukere med samme offentlige IP, og NAT-porten er tilfeldig og varierende. Derfor er det heller cookies eller session-data i nettleseren som automatisk sendes sammen med dataen som identifiserer brukeren.
![image](https://github.com/user-attachments/assets/4354f9f3-bfe9-4249-8920-1b59b7943dc5)


## TCP vs UDP
TCP er stateful, som i praksis betyr at etter et handshake vil en forbindelse være åpen til den lukkes og må evt åpnes på nytt. Det er NAT-tabellen som holden forbindelsen åpen tror jeg. UDP er stateless - bare enkeltpakker fram og tilbake.

## Om prioritering av nettverkskort
vEthernet (og andre nettverkskort) kan sette seg foran det kortet du egentlig bruker for å nå internett. For å se prioritering:
'' Get-NetIPInterface | Sort-Object InterfaceMetric | Format-Table InterfaceAlias, AddressFamily, InterfaceMetric

Og så endre prioritering. Eksempel med kort "Wi-Fi":
'' Set-NetIPInterface -InterfaceAlias "Wi-Fi" -InterfaceMetric 4

## Om virtuelle disker
Merk at virtuelle disker ikke slettes automatisk, selv om du sletter tilhørende VM. Du kan se hvor de lagres når du oppretter en VM - gå til den mappen og slett eventuelle gamle disker. Disse kan fort ta en del plass.

## Løst
* Ved å kjøpe et domenenavn får du rett til å konfigurere hvilken IP-adresse domenet skal peke til i den globale DNS-en (på internett). På ditt eget, lokale nettverk kan du i teorien bruke hvilket domenenavn du vil, uten å kjøpe det.
    * Hvis du for eksempel kjøper et .no-domene, kan du via en registrar (som Domeneshop) fortelle Norid hva domenet faktisk peker til.


Maskiner logges inn i AD for å kunne styres og sikres sentralt.
Brukere logges inn i AD for å kunne få personlig tilgang, sikkerhet og rettigheter.


"GPO-er linkes til en OU, men policyer som handler om brukertillatelser (som login) må referere til grupper, ikke OUs." wtf is this shit...


//////
Fordi brukerkontoen ligger lagret hos domenekontrollen kan en domenebruker logge seg inn på alle maskiner i domenet. 

"Rettighetene domenebrukerkontoen har, reguleres av domenekontrolleren og kan variere fra domeneadministratorer som har alle rettigheter over alle maskiner og ressurser i domenet, til engangs gjestebrukerkontoer som bare kan åpne spesifikke programmer på en spesifikk datamaskin." (NDLA)

...