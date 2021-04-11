# Introduzione

La GGRasp nasce per creare UPnP Renderer attraverso una raspberry per fa suonare la radio antica della Nonna Tina,
poi si evolve come server per l'automazione della casa in cui viene ospitato Home Assistant tramite Docker.
Inoltre viene collegata un'interfaccia per accedere al bus domotico By-me per l'integrazione di servizi personalizzati.

# Connessione

```
ssh pi@ggrasp.station
```

# Installazione OS

## Prima versione con Raspbian stretch e raspberry Pi 1 e 2

* download `2017-11-29-raspbian-stretch-lite`
* download `etcher-1.2.1-linux-x86_64`
* eseguito `etcher-1.2.1-linux-x86_64` e creata una scheda SD con raspbian stretch-lite

* inserita SD su PC e creato un file vuoto nella partizione di boot della scehda SD chiamato `ssh` per abilitare l'accesso via ssh alla board

* inserita SD su board

* connessione da PC tramite ssh utilizzando hostname della board:

  ```
  ssh pi@raspberrypi.lan
  ```

* eseguito da board: `sudo raspi-config`
 * cambiato `hostname` in `ggrasp`
 * cambiato `locales`
 * abilitato audio all'avvio
 * cambiata psw di default
 * disabilitato terminale su seriale ma abilitata la seriale (futura connessione TP-UART)
 * cambiato timezone

* abilitati alias `ll` su `.bashrc`

* aggiunta chiave pubblica del PC sulla board per connessione senza psw


## Seconda versione con Raspbian buster e Raspberry Pi 3

* download `2021-03-04-raspios-buster-armhf-lite.zip`
* utilizzo `imager` per creare la uSD
* creato file `ssh` sulla partizione di boot per avviare il servizio SSH
* abilitata console via seriale:
  * nel file `cmdline.txt` abilitare la console via seriale `console=serial0,115200`
  * nel file `config.txt` abilitare la seriale `enable_uart=1`
* collegarsi via seriale con utenze di default `pi` e password `raspberry`
* eseguire `sudo raspi-config`
  * impostare il Wi-Fi
  * cambiare hostname in `ggrasp`
  * cambiare `locales`
  * abilitato audio all'avvio
  * cambiata psw di default
  * disabilitato terminale su seriale ma abilitata la seriale (futura connessione TP-UART)
  * cambiare timezone
* aggiunta chiave pubblica PC: `ssh-copy-id pi@ggrasp.station`


# Installazione applicativi

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install raspi-config wiringpi
sudo apt-get install zsh vim tmux htop lnav dialog ncdu silversearcher-ag exuberant-ctags avahi-daemon dosfstools minicom rsync git fonts-powerline  nfs-kernel-server
sudo apt-get install build-essential autotools-dev autoconf automake autoconf-archive gnu-standards autoconf-doc libtool libcmocka0 libcmocka-dev libhidapi-hidraw0 libhidapi-dev udev cmake cgdb libncurses5-dev libcpputest-dev
```

* setup DOTFILES: https://github.com/giursino/dotfiles
* sincronizzazione HOME precedente
* modifica `/etc/motd`
* setup CRON con `my-crontab`
* setup NFS server `/etc/exports`


## Installazione applicativi audio

[Riferimento UPnP Renderer.](https://www.lesbonscomptes.com/pages/raspmpd.html)

* test funzionamento jack audio:

  ```
  $ aplay /usr/share/sounds/alsa/Front_Center.wav
  $ speaker-test -t sine -f 440 -c 2 -s 1
  ```

* installato MPD: `sudo apt-get install mpd`

  Su RPi3 ho avuto problemi con il default di MPD:

  * cambiato settaggio di `/var/log/mpd/mpd.log` con utente `mpd` e gruppo `audio`
  * inserito `mpd` in gruppo `audio`
  * cambiato file di configurazione di MPD:
  
    ```
    audio_output {
        type            "alsa"
        name            "My ALSA Device"
        device          "hw:0,0"
        mixer_control   "Headphone"
    }
    ```

* installato UPMPDCLI:
  
  ```
  sudo nano /etc/apt/sources.list.d/upmpdcli.list
  -- aggiunta riga
  deb http://www.lesbonscomptes.com/upmpdcli/downloads/raspbian/ buster main

  sudo apt-get update
  sudo apt-get install upmpdcli

* modificato `/etc/upmpdcli.conf` in modo da impostare la scheda di rete da usare `upnpiface = wlan0`

* modificato `/usr/share/upmpdcli/icon.png` e altri per puntare ai rispettivi file sulla home

* riavviato e provato controllo da telefono tramite BubbleUpnp

## Docker

Per l'installazione seguire le indicazioni ufficiali sul sito ed eseguire

```
get-docker.sh
```

Lo script installerà il sorgente APT, come segue:

```
cat /etc/apt/sources.list.d/docker.list 
deb [arch=armhf] https://download.docker.com/linux/raspbian stretch stable
```

