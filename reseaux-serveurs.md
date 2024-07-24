# les réseaux et les serveurs lorawan

Nous avons le choix entre mettre en place toute l'infrastructure du réseau ou nous en remettre à un opérateur. Il existe trois possibilités pour l'architecture LoRaWAN.

- 🏢 Nous pouvons faire appel à des opérateurs publics qui disposent de réseaux opérationnels à l'échelle nationale.
- 🌐 Nous pouvons construire notre propre réseau LoRaWAN privé.
- 🔄 Nous pouvons construire un réseau hybride en ne mettant en place qu'une partie de l'infrastructure.(réseau dédié avec the things network par exemple)


## Les réseaux LoRaWAN d'opérateurs publics

<img width="769" alt="operateur-public" src="https://github.com/user-attachments/assets/bf28fc2b-2ddb-49f5-bf69-1434ca41d1fc">

- L'utilisateur souscrit à un ou plusieurs forfaits pour pouvoir connecter sa flotte de Device. À titre d'exemple, voici les abonnements proposés par Objenious et Orange en 2022 pour avoir accès à leur réseau LoRaWAN :

### Orange :
- 📡 Uplink illimité (dans le respect du Duty-cycle).
- 💬 Le prix de chaque message downlink est de 5 cts.
- 💳 L'abonnement varie de 1€/mois (36 mois) à 2€/mois (sans engagement).  

### Bouygues Objenious :
- 📈 144 messages uplink par jour.
- 📉 6 messages downlink par jour.
- 💰 L'abonnement est de 20 € / an.

## Les réseaux LoRaWAN privés

<img width="647" alt="reseau-prives" src="https://github.com/user-attachments/assets/95abc6fa-d067-4345-87ed-00792514b675">

`La Gateway comprend le Network Server et l’Application Server (ligne continue verte). La plateforme IoT est parfois également incluse (ligne verte en pointillés).`

- Vous devrez mettre en place votre propre Gateway et votre propre infrastructure de serveurs pour communiquer avec vos Devices. Vous devrez également vous occuper de l'administration de votre serveur LoRaWAN.
  
- 📡 Dans certaines Gateway, une instance d'un serveur LoRaWAN est proposée. Cela simplifie l'infrastructure globale car vous avez tout dans un seul boitier (la Gateway) mais cela limitera considérablement le potentiel de votre réseau.

## le réseau LoRaWAN hybride

Il existe une autre possibilité qui est un intermédiaire entre le réseau public et le réseau privé. Elle présente l'avantage de gérer la couverture du réseau en utilisant ses propres Gateways, tout en confiant l'infrastructure du serveur LoRaWAN à un fournisseur de service afin de limiter les investissements et la maintenance.

<img width="754" alt="reseau-dedié" src="https://github.com/user-attachments/assets/bd599a1b-0dfb-40f7-821d-8c5328a01def">

`La Gateway comprend le Network Server et l’Application Server (ligne continue verte). La plateforme IoT est parfois également incluse (ligne verte en pointillés).`

- Mettre en place votre propre Gateway et votre propre infrastructure de serveurs pour communiquer avec vos Devices. Vous devrez également vous occuper de l'administration de votre serveur LoRaWAN.

- 📡 Dans certaines Gateway, une instance d'un serveur LoRaWAN est proposée. Cela simplifie l'infrastructure globale car vous avez tout dans un seul boitier (la Gateway) mais cela limitera considérablement le potentiel de votre réseau.


