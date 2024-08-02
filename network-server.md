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
1. Add device profile

-> Une fois le device profile enregistré je vais pouvoir aller dans mon application et créer un device

NB :  Suivre le fil d'ariane au niveau de la partie headers
file d'ariane Tenants/ChirpStack/Applications/myApplication

2. Add device
Entrer un nom , une description , deviceEUI doit etre celui entré dans le device OTAA que l'on veut activer, enfin je dois lui assigner un device profile que l'on a enregistrer precedement.

Si cela n'est pas encore fait , configurer au prealable le micro controller.
Dans mon cas a moi c'est un **Lora e5 mini model STM32WLE5JC**
Voir doc: https://wiki.seeedstudio.com/LoRa_E5_mini/

Dans une console de serie (ex : arduino ide ou plateformeio )
Brancher le micro controller a la prise usb et ensuite tapper les commandes suivantes format textedans la console afin d'obtenir
les informations nécessaires à l'enregistrement par OTAA de la device :
EXEMPLE:
Tx: AT+ID=DevEui
Rx: +ID: DevEui, 2C:F7:F1:20:24:90:03:63
Tx: AT+ID=AppEui
Rx: +ID: AppEui, 80:00:00:00:00:00:00:07
Tx: AT+KEY=APPKEY,"2B7E151628AED2A6ABF7158809CF4F3C"
Rx: +KEY: APPKEY 2B7E151628AED2A6ABF7158809CF4F3C


-> Submit

Et voila la nouvelle device est créée.
La configuration du fil d'ariane est la suivante:
Tenants/ChirpStack/Applications/myApplication/Devices/myDeviceOTAA


