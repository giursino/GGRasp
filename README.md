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
* connessione da PC tramite ssh utilizzando hostname della board: `ssh pi@raspberrypi.lan`
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

* impostare la chiave ssh da usare per github
  * copiarla
  * oppure rigenerarla `ssh-keygen -t ed25519 -C "pi@ggrasp.station` e poi inserirla su github
* setup DOTFILES:  `git clone git@github.com:giursino/dotfiles`
* cambiare shell `chsh -s /bin/zsh`
* clone repo GGRasp:  `git clone git@github.com:giursino/GGRasp.git`
* zsh: `sudo ln -sf /home/pi/GGRasp/customs/zsh/zshrc.local /home/pi/.zshrc.local`
* motd: `sudo ln -sf /home/pi/GGRasp/customs/motd/motd /etc/motd`
* cron.d:
  * copia `sudo cp /home/pi/GGRasp/customs/cron.d/my-crontab /etc/cron.d/my-crontab`
  * link nella home: `ln -s /etc/cron.d/my-crontab /home/pi/my-crontab`
* NFS server: `sudo ln -sf /home/pi/GGRasp/customs/nfs/exports /etc/exports`
* sincronizzazione HOME precedente

## Abilitato filesystem in readonly

Utilizzare il metodo `overlayfs` fornito dal kernel per gestire il sistema in readonly, in modo da preservare la uSD.

Visto che il metodo `overlay` non è ancora supportato nativamente dalla version 241 di `systemd`, ma solo dalla 245,
avevo iniziato a creare degli script per utilizzare il modulo `overlay`; poi ho scoperto che il tool `raspi-config`
mette a disposizione la feature, quindi l'ho usata.
Ho migliorato la feature inserendo il tool [`overctl`](https://yagrebu.net/unix/rpi-overlay.md) e il `motd` per mostrare
all'accesso com'è montato il file-system.

Ho deciso di tenere in read-only solo la parte di SO non i dati utenti, quindi ho partizionato la uSD così:
```
Device         Boot    Start      End  Sectors  Size Id Type
/dev/mmcblk0p1          8192   532479   524288  256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2        532480 16916479 16384000  7,8G 83 Linux
/dev/mmcblk0p3      16916480 62332927 45416448 21,7G 83 Linux
```

La HOME è montata sulla partizione 3 dei dati

### Riferimenti
* [Documentazione iniziale](https://yagrebu.net/unix/rpi-overlay.md)
* [Repo github contente i files](https://github.com/ghollingworth/overlayfs)
* [Repo alternativo](https://github.com/JasperE84/root-ro)
* [Lungo thread di RPi dov'è partito tutto](https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=161416)
* [Utilizzo vecchio metodo senza overlay](https://hallard.me/raspberry-pi-read-only/)

### Step da seguire

* partizionare la uSD aggiungendo la partizione 3 utilizzando `gparted` da un PC con la uSD inserita
* avviare il sistema
* disabilitare lo swap
  ```
  sudo dphys-swapfile swapoff
  sudo dphys-swapfile uninstall
  sudo update-rc.d dphys-swapfile disable
  sudo systemctl disable dphys-swapfile
  ```
* montare la partizione dei dati temporaneamente e creare la cartelle utente
  ```
  sudo mkdir /tmp/home
  sudo mount /dev/mmcblk0p3 /tmp/home
  sudo cp -a /home/. /tmp/home/
  sudo umount /tmp/home
  ```
* modificare `fstab` per montare la home sulla partizione dedicata
  ```
  proc                  /proc           proc    defaults          0       0
  PARTUUID=ab962e14-01  /boot           vfat    defaults          0       2
  PARTUUID=ab962e14-02  /               ext4    defaults,noatime  0       1
  PARTUUID=ab962e14-03  /home           ext4    defaults,noatime  0       2
  ```
* copiare `overctl`: `sudo cp /home/pi/GGRasp/customs/overlayfs/overctl /usr/local/sbin`
* modificare motd: `sudo ln -sf /home/pi/GGRasp/customs/overlayfs/80-overlay /etc/update-motd.d`
* attivare read-only filesystem su `sudo raspi-config`
  * Performance > Overlay File System
  * Non bloccare `/boot`
* riavviare


Verificare che il metodo funzioni:

* eseguire letture su uSD: `cat /sys/fs/ext4/mmcblk0p2/lifetime_write_kbytes`
* verificare la tipologia di mount tramite il comando `findmnt`:
  * read-write:
    ```
    TARGET                                SOURCE         FSTYPE     OPTIONS
    /                                     /dev/mmcblk0p2 ext4       rw,noatime
  * read-only:
    ```
    TARGET                                SOURCE         FSTYPE     OPTIONS
    /                                     /dev/mmcblk0p2 ext4       rw,noatime
    ```


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
  ```
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

### Docker su ambiente overlayfs

Nel caso si usi un ambiente in read-only basato su `overlayfs`, Docker
non funzionerà perchè utilizza anch'esso il modulo `overlayfs` e da
specifica non è possibile montare una cartella in overlayfs su una
cartella la cui radice è già montata in overlayfs.

Pertanto è necessario spostare la home di docker su un filesystem non
montato con overlayfs.

Nel mio caso, ho deciso di spostare la cartella di lavoro di docker
`/var/lib/docker` sulla cartella `/home/docker` che è montata in RW.

* Stop docker:
  ```
  sudo systemctl stop docker
  sudo systemctl stop docker.socket
  ```

* Copiare contenuto della cartella di lavoro di Docker nella nuova
  posizione:
  ```
  sudo rsync -a /var/lib/docker /home
  ```
* Cambiare cartella di lavoro, creando il file
  `/etc/docker/daemon.json`:
  ```
  {
    "data-root": "/home/docker"
  }
  ```

* Riavviare docker:
  ```
  systemctl daemon-reload
  systemctl restart docker
  ```

* Verificare aggiornamento root:
  ```
  docker info
  ```

## KNXD

Ho installato knxd in modo da accedere al bus tramite una interfaccia
KNX USB. Ho utilizzato questo servizio per eseguire l'integrazione con
HomeAssistant.

### Installazione

```
sudo apt-get install knxd knxd-tools
```

### Setup

Modificare il file di configurazione `/etc/knxd.conf` in modo da
utilizzare il file `knxd.ini` presente nell'home.

```
/etc/knxd.conf:
  KNXD_OPTS=/home/pi/knxd.ini
```

Aggiungere al file di configurazione di `udev` l'interfaccia KNX USB

```
/usr/lib/udev/rules.d/60-knxd.rules:
  ATTR{idVendor}=="16d0", ATTR{idProduct}=="0490",OWNER="knxd",MODE="0600"
  ATTR{idVendor}=="28c2", ATTR{idProduct}=="001a",OWNER="knxd",MODE="0600"
```

Riavviare il servizio

```
systemctl restart knxd.service
```

Verificare sia attivo guardando i log di sistema o che sia presente
la porta TCP 6666

```
$ netstat -tln
...
tcp        0      0 0.0.0.0:6666            0.0.0.0:*               LISTEN
...
```

## Home Assistant

Ho installato Home Assistant Core su container Docker.
Mancando il Supervisor ho installato successivamente HACS in modo da
ottenere i plugin anche su ambiente Core.

* [Riferimento su indomus](https://indomus.it/guide/come-installare-e-configurare-home-assistant-core-con-docker-su-un-raspberry-pi-gia-in-uso/).
* [Riferimento ufficiale](https://www.home-assistant.io/installation/raspberrypi#install-home-assistant-container)

### Comandi

**primo avvio**:

```
docker run --init -d \
  --name homeassistant \
  --restart=unless-stopped \
  -v /etc/localtime:/etc/localtime:ro \
  -v /home/pi/hass:/config \
  --network=host \
  homeassistant/raspberrypi3-homeassistant:stable
```

**riavvio**: `docker restart homeassistant`

**accesso alla shell**: `docker exec -it homeassistant bash`

### Aggiornamento

[](https://www.home-assistant.io/common-tasks/container/)

```
docker pull ghcr.io/home-assistant/raspberrypi3-homeassistant:stable
docker stop homeassistant
docker rm homeassistant
docker run -d \
  --name homeassistant \
  --restart=unless-stopped \
  --privileged \
  -e TZ=Europe/Rome \
  -v /home/pi/hass:/config \
  --network=host \
  ghcr.io/home-assistant/raspberrypi3-homeassistant:stable
```

