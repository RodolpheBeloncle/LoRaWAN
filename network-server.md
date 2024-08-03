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
Néanmoins on a la possibilité d'en créer d'autres depuis l'onglet <Tenants>

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

Récuperer alors l'identifiant unique à chaque gateway qui est inscrit dans la partie configuration.
 
L'enregistrement de la gateway est terminé, on peut passer à l'étape 5 afin de configurer la gateway.

#### 5. Configuration de la gateway

Changer la configuration actuelle afin que la gateway depuis le réseau local puisse envoyer les datas au network server


<img width="914" alt="configuration" src="https://github.com/user-attachments/assets/75633e97-e4d1-4572-8e8a-18bc8732fb2a">

#### 6. Enregistrement de la device stm32 lora Wio-e5 mini

Vérification des Clés
Assurez-vous que les clés configurées sur le dispositif STM32 correspondent exactement à celles enregistrées dans ChirpStack.

-> **Clés sur ChirpStack**
AppEUI: 52:69:73:69:6E:67:48:46
DevEUI: 2C:F7:F1:20:51:00:49:15
AppKey: (Doit être vérifiée dans l'interface ChirpStack)

-> **Clés sur le Dispositif STM32**
Vérifiez les valeurs actuelles configurées sur votre STM32:
AT+ID=DevEUI
AT+ID=AppEUI
AT+KEY=APPKEY

-> **Configuration des Clés sur STM32**

Configurer le DevEUI:
AT+ID=DevEUI,"2CF7F12051004915"

Configurer l'AppEUI:
AT+ID=AppEUI,"526973696E674846"

Configurer l'AppKey:
Assurez-vous que l'AppKey configuré dans le dispositif STM32 correspond exactement à celui enregistré dans ChirpStack.

AT+KEY=APPKEY,"<votre AppKey>"
Vérification et Configuration Régionale
Configurer la région:

AT+DR=EU868
Configurer le mode OTAA:

AT+MODE=LWOTAA
Tentative de Rejoindre le Réseau
Envoyer la commande de join:

AT+JOIN

Enfin si vous retoruner sur l'interface chirstack, cela devrait ressembler à ca :

![interface-chirpstack](https://github.com/user-attachments/assets/1dcf7cd9-c25f-449d-9b27-54c45e1791db)

La gateway et la device son activés

-> **Envoie de données en queue**
![enqueur-message](https://github.com/user-attachments/assets/1b257d8b-183b-4d14-9960-8488147b43e9)
Rappel: device classe A pour que je puisse recevoir une donnée, il faut que j'emette un flux uplink.

Enqueue data : flux downlink AQ==
test : flux uplink -> AT+MSG="Data to send"


-> **Troubleshooting**
 - Vérification des ports et de l'adresse ip de l'Ébergeur du service worker
 - Activer les logs : AT + LOG=ON
 - Utilisation de l'application wirechark pour verifier les fluxs

-> **Intégration d'application via mqtt**

Modifier la configuration dans le fichier chirpstack-docker > configuration > chirpstack > chirpstack.toml
NB : Utiliser mqtt-explorer comme client
Se connecter en local 127.0.0.1:1883

<img width="844" alt="mqtt-client" src="https://github.com/user-attachments/assets/933967f2-fb63-440f-8cdc-c16c95c70b89">
1. subscribe topic uplink: application/a52fe825-8112-401f-ae14-dffc00b55322/device/2cf7f12051004915/event/up

2. publisher topic downlink: application/a52fe825-8112-401f-ae14-dffc00b55322/device/2cf7f12051004915/command/down
