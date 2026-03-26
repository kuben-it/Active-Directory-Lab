### No computer account
![image](https://github.com/user-attachments/assets/ad385cb9-e4a9-4553-b856-4652d0b32fd1)

Dette betyr at klient-maskinen ikke er registrert i domenet. Hvis du får dette fra en klient må du bare melde inn klienten. Hvis du får det på domenekontrolleren - ånei! Håper du har en checkpoint.

### Bootable media not found elns
Høyreklikk på maskinen > Settings > DVD Drive > Image File: velg ISO-filen du skal installere

![image](https://github.com/user-attachments/assets/b71db705-2ea7-49bd-9b96-832a032cf7b7)

### Feil ved innmelding av klient
Hvis du får en feil som "An Active AD DC for the domain could not be contacted" kan du:
* Sjekk at DC kjører og fungerer, inkludert DHCP
* Det kan hende at klienten ikke fant DC da den ble skrudd på og derfor ikke fikk IP fra DHCPen. Kjør "ipconfig" i CMD - hvis du ikke har fått en IP i rangen du definerte i DHCP kan du prøve å kjøre "ipconfig /release" og så "ipconfig /renew" for at klienten skal se etter DHCP-serveren på nytt. 

### Pinging fungerer bare en vei
Streng brannmur? Win + R > wf.msc for å åpne brannmur-innstillinger

Hvis du ikke får lov til å åpne brannmur på domenebruker kan du logge inn som lokal admin. Da må du gå på Other Users og skrive inn .\ før administrator-navnet.

Under "Inbound Rules" finn regelen som heter:
    File and Printer Sharing (Echo Request - ICMPv4-In)
    Finn to versjoner av regelen (for Private og Public)
    Høyreklikk på begge og velg Enable Rule