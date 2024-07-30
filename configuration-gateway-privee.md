# Configuration gateway raspberry

voir plus : 
https://www.thethingsindustries.com/docs/gateways/models/raspberry-pi/


# Démarrer et Configurer votre Raspberry Pi

## Introduction

Dans ce guide, nous allons voir comment démarrer et configurer votre Raspberry Pi, notamment pour utiliser LoRa Basics™ Station. Suivez ces étapes pour préparer votre Raspberry Pi et installer les logiciels nécessaires.

## Étape 1 : Démarrage de votre Raspberry Pi

Insérez la carte SD dans votre Raspberry Pi et allumez-le.

### Connexion Physique

Si vous n'avez pas activé l'accès SSH, vous devrez connecter un écran et un clavier externes pour le configurer.

### Connexion à Distance

Si vous avez activé SSH à l'étape précédente, vous pouvez vous connecter à votre Raspberry Pi à distance depuis votre ordinateur. Pour cela, trouvez l'adresse IP de votre Raspberry Pi en listant les appareils sur votre réseau local (par exemple avec `nmap`) et connectez-vous via SSH. Le nom d'utilisateur par défaut est `pi` et le mot de passe par défaut est `raspberry`.

```bash
ssh pi@192.168.1.2
```
### Mise à Jour des Paquets Système
Après vous être connecté à votre Raspberry Pi, mettez à jour les paquets système aux dernières versions :

```bash

sudo apt-get update
sudo apt-get upgrade

```
### Activer l'Interface SPI

Pour activer l'interface SPI, utilisez l'outil raspi-config :

```bash

sudo raspi-config
```
Un assistant raspi-config apparaîtra. Utilisez les touches fléchées et la touche Entrée pour naviguer. Choisissez Interface Options → SPI, puis sélectionnez pour activer l'interface SPI. Après l'avoir activée, appuyez sur la touche Échap pour quitter l'outil raspi-config.

## Étape 2 : Installer les Paquets Nécessaires
Installez les paquets nécessaires pour construire le packet forwarder :

```bash

sudo apt-get install git gcc make
```
## Étape 3 : Construire le Packet Forwarder LoRa Basics™ Station
Cloner le Dépôt
Clonez le dépôt The Things Stack :

```bash

git clone https://github.com/lorabasics/basicstation
```
### Construire le Binaire
Construisez le binaire de la station LoRa Basics™ :

```bash

cd basicstation
make platform=rpi variant=std ARCH=$(gcc --print-multiarch)
```
### Vérifier la Construction
Assurez-vous que le binaire a été construit avec succès :

```bash

./build-rpi-std/bin/station --version
```
Vous devriez voir quelque chose comme ceci :

```yaml

Station: 2.0.6(rpi/std) 2022-03-26 17:43:16
Package: (null)
```

### Installer le Binaire
Installez le binaire de la station LoRa Basics™ :

```bash

sudo mkdir -p /opt/ttn-station/bin
sudo cp ./build-rpi-std/bin/station /opt/ttn-station/bin/station
```

## Étape 4 : Dériver l'EUI de la Passerelle
Pour dériver l'EUI de la passerelle, utilisez une combinaison de l'adresse MAC de la passerelle et de FFFE, comme suit :

```bash

export MAC=`cat /sys/class/net/eth0/address`
export EUI=`echo $MAC | awk -F: '{print $1$2$3 "fffe" $4$5$6}'`
echo "The Gateway EUI is $EUI"
```
Le résultat sera quelque chose comme :

```bash

The Gateway EUI is b827ebfffee00c83
```
Assurez-vous de noter cela pour les étapes suivantes.

## Étape 5 : Enregistrer la Passerelle sur The Things Stack
Pour enregistrer votre passerelle sur The Things Stack, suivez le processus décrit dans la section Adding Gateways. Pour l'EUI de la passerelle, utilisez l'EUI dérivé à l'étape précédente.
https://www.thethingsindustries.com/docs/gateways/concepts/adding-gateways/

Il est également recommandé d'activer l'option Require authenticated connection lors de l'enregistrement.

## Étape 6 : Créer une Clé API
Ensuite, vous devez créer une clé API pour votre passerelle sur The Things Stack, qui sera utilisée pour l'authentification de votre passerelle.

Suivez les instructions dans la section LNS pour créer une clé API avec les droits Link as Gateway to a Gateway Server for traffic exchange, c'est-à-dire write uplink et read downlink. Assurez-vous de copier la clé car vous ne pourrez pas la voir à nouveau.

## Étape 7 : Configurer LoRa Basics™ Station
L'étape suivante consiste à créer les fichiers de configuration nécessaires pour que la passerelle LoRa Basics™ Station se connecte à The Things Stack.

### Créer un Répertoire de Configuration
Sur votre Raspberry Pi, créez un nouveau répertoire :

```bash

sudo mkdir -p /opt/ttn-station/config
```

### Fichier de Configuration tc.uri
Créez un fichier de configuration tc.uri contenant une adresse de serveur LNS. Par exemple, si vous utilisez le cluster eu1 de The Things Stack Sandbox :

```bash

echo 'wss://eu1.cloud.thethings.network:8887' | sudo tee /opt/ttn-station/config/tc.uri
```
### Fichier de Configuration tc.key
Créez le fichier de configuration tc.key contenant un en-tête d'autorisation. Cet en-tête contiendra la clé API que vous avez créée à l'étape précédente et sera utilisé pour authentifier la connexion de votre passerelle.

```bash

export API_KEY="NNSXS.XXXXXXXXXXXXXXX.YYYYYYYYYYYYYYYY"
echo "Authorization: Bearer $API_KEY" | perl -p -e 's/\r\n|\n|\r/\r\n/g' | sudo tee -a /opt/ttn-station/config/tc.key
```
### Fichier de Configuration tc.trust
Créez le fichier de configuration tc.trust qui sera le CA racine utilisé pour vérifier les certificats de votre serveur LNS. Vous pouvez utiliser les certificats CA du système :

```bash

sudo ln -s /etc/ssl/certs/ca-certificates.crt /opt/ttn-station/config/tc.trust

```
### Fichier de Configuration station.conf
Enfin, créez le fichier de configuration station.conf contenant les options de configuration pour votre concentrateur.

```bash
echo '
{
    /* If slave-X.conf present this acts as default settings */
    "SX1301_conf": { /* Actual channel plan is controlled by server */
        "lorawan_public": true, /* is default */
        "clksrc": 1, /* radio_1 provides clock to concentrator */
        /* path to the SPI device, un-comment if not specified on the command line e.g., RADIODEV=/dev/spidev0.0 */
        /*"device": "/dev/spidev0.0",*/
        /* freq/enable provided by LNS - only HW specific settings listed here */
        "radio_0": {
            "type": "SX1257",
            "rssi_offset": -166.0,
            "tx_enable": true,
            "antenna_gain": 0
        },
        "radio_1": {
            "type": "SX1257",
            "rssi_offset": -166.0,
            "tx_enable": false
        }
        /* chan_multiSF_X, chan_Lora_std, chan_FSK provided by LNS */
    },
    "station_conf": {
        "routerid": "B827EBFFFEA3A5C3",
        "log_file": "stderr",
        "log_level": "DEBUG", /* XDEBUG,DEBUG,VERBOSE,INFO,NOTICE,WARNING,ERROR,CRITICAL */
        "log_size": 10000000,
        "log_rotate": 3,
        "CUPS_RESYNC_INTV": "1s"
    }
}
' | sudo tee /opt/ttn-station/config/station.conf
```
Enfin, créez le script start.sh, qui sera utilisé pour réinitialiser l'iC880A via son pin de réinitialisation et démarrer le transfert de paquets 

```bash

echo '#!/bin/bash

# Reset iC880a PIN
SX1301_RESET_BCM_PIN=25
echo "$SX1301_RESET_BCM_PIN"  > /sys/class/gpio/export
echo "out" > /sys/class/gpio/gpio$SX1301_RESET_BCM_PIN/direction
echo "0"   > /sys/class/gpio/gpio$SX1301_RESET_BCM_PIN/value
sleep 0.1
echo "1"   > /sys/class/gpio/gpio$SX1301_RESET_BCM_PIN/value
sleep 0.1
echo "0"   > /sys/class/gpio/gpio$SX1301_RESET_BCM_PIN/value
sleep 0.1
echo "$SX1301_RESET_BCM_PIN"  > /sys/class/gpio/unexport

# Test the connection, wait if needed.
while [[ $(ping -c1 google.com 2>&1 | grep " 0% packet loss") == "" ]]; do
echo "[TTN Gateway]: Waiting for internet connection..."
sleep 30
done

# Start station
/opt/ttn-station/bin/station
' | sudo tee /opt/ttn-station/bin/start.sh

```
Vous pouvez verifier si le script est executable
```bash
sudo chmod +x /opt/ttn-station/bin/start.sh

```

## Test du Packet Forwarder
```bash
cd /opt/ttn-station/config
sudo RADIODEV=/dev/spidev0.0 /opt/ttn-station/bin/start.sh
```

Cela initialisera la carte concentrateur, connectera votre passerelle à The Things Stack, récupérera la configuration en fonction de votre plan de fréquence et commencera à écouter les paquets. Si vous remarquez quelque chose comme :

```bash
2022-03-27 02:11:50.009 [S2E:VERB] RX 867.5MHz DR5 SF7/BW125 snr=6.2 rssi=-103 xtime=0xE0000000E34FB4 - updf mhdr=40 DevAddr=260B0748 FCtrl=00 FCnt=35 FOpts=[] 014A mic=1717970429 (14 bytes)
2022-03-27 02:12:06.130 [S2E:VERB] RX 867.3MHz DR5 SF7/BW125 snr=8.0 rssi=-102 xtime=0xE0000001D92CCB - updf mhdr=40 DevAddr=260B0748 FCtrl=00 FCnt=36 FOpts=[] 01EA mic=463407879 (14 bytes)
```
## Exécuter le Packet Forwarder en tant que Service Système

La dernière étape consiste à configurer le packet forwarder pour qu'il s'exécute en tant que service système sur le Raspberry Pi. Cela garantit que le forwarder démarrera automatiquement après le démarrage du Raspberry Pi.

Tout d'abord, créez le fichier de configuration du service systemd :

```sh
echo '
[Unit]
Description=The Things Network Gateway

[Service]
WorkingDirectory=/opt/ttn-station/config
ExecStart=/opt/ttn-station/bin/start.sh
SyslogIdentifier=ttn-station
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
' | sudo tee /lib/systemd/system/ttn-station.service

```
Activez le service avec :

```bash
sudo systemctl enable ttn-station

```
Démarrez le service :

```bash
sudo systemctl start ttn-station

```
Vous pouvez observer les journaux du packet forwarder en utilisant la commande suivante :

```bash
sudo journalctl -f -u ttn-station
```
**Voilà ! Votre passerelle est maintenant entièrement fonctionnelle et vous pouvez commencer à développer votre cas d'utilisation IoT.**
