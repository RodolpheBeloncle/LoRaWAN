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
