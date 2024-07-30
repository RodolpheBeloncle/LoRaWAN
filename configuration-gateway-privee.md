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

Créer un Répertoire de Configuration
Sur votre Raspberry Pi, créez un nouveau répertoire :

```bash

sudo mkdir -p /opt/ttn-station/config
```

Fichier de Configuration tc.uri
Créez un fichier de configuration tc.uri contenant une adresse de serveur LNS. Par exemple, si vous utilisez le cluster eu1 de The Things Stack Sandbox :

```bash

echo 'wss://eu1.cloud.thethings.network:8887' | sudo tee /opt/ttn-station/config/tc.uri
```
Fichier de Configuration tc.key
Créez le fichier de configuration tc.key contenant un en-tête d'autorisation. Cet en-tête contiendra la clé API que vous avez créée à l'étape précédente et sera utilisé pour authentifier la connexion de votre passerelle.

```bash

export API_KEY="NNSXS.XXXXXXXXXXXXXXX.YYYYYYYYYYYYYYYY"
echo "Authorization: Bearer $API_KEY" | perl -p -e 's/\r\n|\n|\r/\r\n/g' | sudo tee -a /opt/ttn-station/config/tc.key
```
Fichier de Configuration tc.trust
Créez le fichier de configuration tc.trust qui sera le CA racine utilisé pour vérifier les certificats de votre serveur LNS. Vous pouvez utiliser les certificats CA du système :

```bash

sudo ln -s /etc/ssl/certs/ca-certificates.crt /opt/ttn-station/config/tc.trust

```
Fichier de Configuration station.conf
Enfin, créez le fichier de configuration station.conf contenant les options de configuration pour votre concentrateur.
