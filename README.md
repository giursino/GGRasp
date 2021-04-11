# Introduzione

Creazione UNP Renderer attraverso una raspberry

Rif: [https://www.lesbonscomptes.com/pages/raspmpd.html]

# Connessione

ssh pi@ggrasp.lan


# Installazione OS

* download 2017-11-29-raspbian-stretch-lite
* download etcher-1.2.1-linux-x86_64
* eseguito etcher-1.2.1-linux-x86_64 e creata una scheda SD con raspbian stretch-lite

* inserita SD su PC e creato un file vuoto nella partizione di boot della scehda SD chiamato "ssh" per abilitare l'accesso via ssh alla board

* inserita SD su board

* connessione da PC tramite ssh utilizzando hostname della board: 
ssh pi@raspberrypi.lan

* eseguito da board: sudo raspi-config
 * cambiato hostname in ggrasp
 * cambiato "locales"
 * abilitato audio all'avvio
 * cambiata psw di default
 * disabilitato terminale su seriale ma abilitata la seriale (futura connessione TP-UART)
 * cambiato timezone

* abilitati alias ll su .bashrc

* aggiunta chiave pubblica del PC sulla board per connessione senza psw


# Installazione applicativi audio

* test funzionamento jack audio: 
$ aplay /usr/share/sounds/alsa/Front_Center.wav
$ speaker-test -t sine -f 440 -c 2 -s 1

* installato MPD: sudo apt-get install mpd

* installato UPMPDCLI:
sudo nano /etc/apt/sources.list.d/upmpdcli.list
    deb http://www.lesbonscomptes.com/upmpdcli/downloads/raspbian-stretch/ stretch main

sudo apt-get update
sudo apt-get install upmpdcli

* riavviato e provato controllo da telefono tramite BubbleUpnp

