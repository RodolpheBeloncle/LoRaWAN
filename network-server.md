# Creation de notre propre network lorawan

## Avec ChirpStack

### Utilisation de Docker mettre en place plus rapidement :
-> Commandes : 

```bash
git clone https://github.com/chirpstack/chirpstack-docker.git
cd chirpstack-docker
docker-compose up

```


### Creation d'une application
NB : une application est déjà crée par défaut qui s'appelle "Chirpstack"
Neanmoins on a la passibilité d'en créer d'autres dans depuis l'onglet <Tenants>

### Création d'un Device Profiles
-> Dans l'onglet <Device Profiles>
  #### 1. Add device profile

-> Une fois le device profile enregistré je vais pouvoir aller dans mon application et créer un device

NB :  Suivre le fil d'ariane au niveau de la partie headers
file d'ariane Tenants/ChirpStack/Applications/myApplication

  #### 2. Add device
Entrer un nom , une description , deviceEUI doit etre celui entré dans le device OTAA que l'on veut activer, enfin je dois lui assigner un device profile que l'on a enregistrer precedement.

Si cela n'est pas encore fait , configurer au prealable le micro controller.
Dans mon cas a moi c'est un **Lora e5 mini model STM32WLE5JC**
Voir doc: https://wiki.seeedstudio.com/LoRa_E5_mini/

Dans une console de serie (ex : arduino ide ou plateformeio )
Brancher le micro controller a la prise usb et ensuite tapper les commandes suivantes au format texte dans la console afin d'obtenir
les informations nécessaires à l'enregistrement par OTAA de la device :

EXEMPLE:

entré
Tx: AT+ID=DevEui
sortie
Rx: +ID: DevEui, 2C:F7:F1:20:24:90:03:63

entré
Tx: AT+ID=AppEui
sortie
Rx: +ID: AppEui, 80:00:00:00:00:00:00:07


-> Submit

Et voila la nouvelle device est créée.
La configuration du fil d'ariane est la suivante:
Tenants/ChirpStack/Applications/myApplication/Devices/myDeviceOTAA

Il va falloir également créer une clé d'application:
Pour la générer taper dans la console de serie:

entré
Tx: AT+KEY=APPKEY,"2B7E151628AED2A6ABF7158809CF4F3C"
sortie
Rx: +KEY: APPKEY 2B7E151628AED2A6ABF7158809CF4F3C

-> Notre device est enfin configurée mais il est inactif,on va pouvoir enregistrer une gateway dans le network server, pour enfin pouvoir voir des 
trames qui partira depuis le device jusque dans l'application server.

#### 4. Enregistrement d'une gateway

Aller dans l'onglet <Gateway>
Add gateway
Preciser uun nom pour la gateway et l'id de la gateway
NB: J'utilise comme gateway un model type PG1302 dragino lorawan concentrateur , qui se branche sur 
un raspberry B.

Voir documentation : 
http://wiki.dragino.com/xwiki/bin/view/Main/User%20Manual%20for%20All%20Gateway%20models/PG1302/#H1.4PinMapping

Si vous avez suivi la documentation l'image flacher sur le raspberry py avec la gateway instancié, on a acces a une interface 
afin de pouvoir configurer la Gateway (passerelle).
 Aller dans un navigateur et tapper : http://IP_ADDRESS

Vous accédez a une interface ressemblant à cela:

![gateway-interface](https://github.com/user-attachments/assets/8877977f-8e8b-4c8e-b95e-67d7537dc82d)

 
