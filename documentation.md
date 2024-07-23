# Système LoRaWAN

 ## Network Server & Application Server

![Network-layout-lorawan](https://github.com/user-attachments/assets/49622f33-e5a9-474d-bd4f-a1b303eb0aae)

## Activation des devices Lora

# Méthodes d'Activation des Dispositifs IoT

- 🔑 **NwSKey (Authentification)** : La clé NwSKey, utilisée pour l'authentification, est gérée par le serveur réseau (Network server).
- 🔒 **AppSKey (Chiffrement)** : La clé AppSKey, utilisée pour le chiffrement des données, est gérée par le serveur d'application (Application server).
- 📇 **DevAddr (Adressage)**: Chaque dispositif final est identifié par une adresse unique appelée DevAddr. Cette adresse est utilisée pour diriger les messages du réseau vers le bon dispositif et pour identifier les dispositifs sur le réseau.

Ces pratiques assurent une séparation claire des responsabilités en matière de sécurité, améliorant ainsi la robustesse globale du système.

## 1. Activation par Personnalisation (ABP)

![ABP-activation](https://github.com/user-attachments/assets/8a9637c6-66de-42ff-9e80-7296435e1f47)


🟢 **Description**  
ABP est une méthode où les clés de session réseau (NwkSKey) et de session d'application (AppSKey) ainsi que l'adresse du dispositif (DevAddr) sont programmées directement dans le dispositif avant le déploiement.  
Pas de processus d'authentification dynamique lors de la mise en ligne du dispositif.

### Avantages
- ✔ **Simplicité** : Pas besoin d'un serveur de jointure.
- ⏩ **Rapide à déployer** : Le dispositif est immédiatement opérationnel une fois alimenté.

### Inconvénients
- ❌ **Moins sécurisé** : Les clés sont fixes et peuvent être compromises.
- ❌ **Moins flexible** : Les modifications des clés nécessitent une reconfiguration physique des dispositifs.

## 2. Activation par Liaison Radio (OTAA)

![otaa-activation](https://github.com/user-attachments/assets/489463d5-0d1f-4ed3-ab8d-4828a7f06e2a)

🔒 **Description**  
OTAA est une méthode où le dispositif envoie une requête de jointure au réseau et reçoit les clés de session (NwkSKey et AppSKey) dynamiquement.  
Processus d'authentification dynamique et sécurisé entre le dispositif et le serveur de jointure.
1. Le device Lora émet un Join-Request à l'aide des informations DevEUI,AppEUI et AppKey qu'il possède.
2. Le Network Server authentifie le Join Request et le valide.Il génère alors une NwkSKEY,UNE appSKey,et un DevAddr.
3. Le Network Server retourne le DevAddr,ainsi qu'une série de paramètres.
4. Les paramètres fournis lors du Join-Accept, associé à l'AppKey permettent au Device Lora de générer le même NwkSKey et le même AppSKey qui avait été initialement généré sur le Network Server.

![OTAA-2](https://github.com/user-attachments/assets/813e2a07-8151-4321-91d1-37fcfdef1921)

⚠️ L'application session key (AppSKey) et L'application key (AppKey) n'ont pas la meme finalité et sont différentes.

### Avantages
- ✔️ **Sécurité accrue** : Les clés sont générées dynamiquement pour chaque session.
- ✔️ **Flexibilité** : Les dispositifs peuvent changer de réseau sans reconfiguration physique.

### Inconvénients
- ❌ **Complexité** : Nécessite un serveur de jointure et un processus de jointure initial.
- ⏳ **Temps d'activation** : Peut prendre plus de temps pour initialiser le dispositif.


 ## Gateways

 ![gateways](https://github.com/user-attachments/assets/04e61e72-e16d-43f7-9ceb-06b1ce2efeac)

 ## Authentification Avec le network server

 ![Authentification-server](https://github.com/user-attachments/assets/dfac128a-30ab-4e2d-8155-a33e0d0db84f)


 # 📶 Classes de Périphériques LoRa

## ⚡ Classe A
**Description :**
Les appareils de classe A sont les plus économes en énergie. Chaque appareil de classe A peut envoyer des messages à tout moment et reçoit les accusés de réception immédiatement après l'émission. Ils ouvrent deux courtes fenêtres de réception après chaque transmission.

**Exemple :**
*Une petite sonde météo alimentée par batterie, envoyant périodiquement des données environnementales.*

 ![Capture d’écran 2024-07-21 à 16 43 14 (2)](https://github.com/user-attachments/assets/47be90f5-61ea-48a6-8222-5a053f3bd748)
---

## 🕒 Classe B
**Description :**
Les appareils de classe B ouvrent des fenêtres de réception supplémentaires à des moments programmés. Ils reçoivent des signaux de synchronisation du réseau pour assurer des intervalles de réception réguliers, ce qui permet une meilleure gestion du temps et de la communication.

**Exemple :**
*Un capteur de parking avec des horaires de communication définis pour optimiser la gestion du stationnement.*

 ![class-b](https://github.com/user-attachments/assets/ecd88543-e412-473f-a12a-7074042aa703)
---

## 🌐 Classe C
**Description :**
Les appareils de classe C sont toujours en mode réception sauf lorsqu'ils transmettent des données. Ils consomment plus d'énergie que les classes A et B, mais offrent une latence plus faible pour les commandes du réseau.

**Exemple :**
*Un système d'éclairage urbain connecté, toujours prêt à recevoir des commandes pour ajuster l'éclairage en temps réel.*

 ![class-c](https://github.com/user-attachments/assets/c18b4380-a082-4bad-ac13-f5d5a051b829)
---

# Attaque par Replay

<img width="784" alt="Capture d’écran 2024-07-23 à 05 18 19" src="https://github.com/user-attachments/assets/8ea88cc5-70b1-49cc-84a0-0d8abd15ab45">

🛡️ **Description**  
Une attaque par replay est une méthode d'attaque où un intrus intercepte et réémet des messages de communication valides pour tromper le système de sécurité. Cela peut permettre à l'attaquant de rejouer les messages interceptés pour obtenir un accès non autorisé ou pour perturber les opérations normales du système.

### Fonctionnement
- 📡 **Interception** : L'attaquant intercepte les messages valides échangés entre deux parties.
- 🔄 **Réémission** : L'attaquant réémet ces messages à une partie du système pour reproduire les actions précédemment autorisées.

### Exemples d'Attaques
- 🏦 **Banques et Paiements** : Réémettre des transactions financières pour effectuer des paiements multiples.
- 🏠 **Systèmes Domotiques** : Réémettre des signaux pour déverrouiller des portes ou activer des systèmes de sécurité.

### Avantages pour l'Attaquant
- 🕵️‍♂️ **Difficulté de Détection** : Les messages réémis sont souvent identiques aux messages légitimes, ce qui rend difficile leur détection.
- 💻 **Simplicité** : Ne nécessite pas de connaissances approfondies pour intercepter et réémettre des messages.

### Contre-mesures
- 🔒 **Chiffrement** : Utilisation de méthodes de chiffrement pour sécuriser les communications.
- 🔁 **Tokens d'Authentification** : Utilisation de tokens uniques ou jetons d'authentification qui expirent après une utilisation.
- ⏲️ **Horodatage** : Incorporer des horodatages dans les messages pour garantir qu'ils ne sont valides que pour une période limitée.
- 🔢 **Compteur de Trames (Frame Counter)** : Utilisation d'un compteur de trames qui incrémente à chaque message envoyé. Si un message avec un compteur inférieur ou égal est reçu, il est rejeté comme étant un possible replay.
  
`=> Principe du Frame counter`: 
*Reception valide seulement si X >= Y*
<img width="670" alt="frame-counter" src="https://github.com/user-attachments/assets/3a3f218e-0044-49f4-a31e-6ccabb2c27d7">

### Inconvénients pour l'Attaquant
- ⏱️ **Temps Limité** : Les contre-mesures comme les horodatages et les tokens d'authentification peuvent limiter la fenêtre d'opportunité pour l'attaque.
- 🛠️ **Complexité Accrue** : Les systèmes avancés de détection et de réponse peuvent compliquer la réussite de l'attaque.

## Adaptive Data Rate (ADR)

Le réglage du SF et du PT (puissance transmise) n'est pas simple. Même si l'on trouve une bonne configuration, la transmission peut être altérée par l'environnement local ou la météo. Pour surmonter cette difficulté, une méthode d'ajustement automatique a été mise en place par le protocole LoRaWAN : il s'agit de l'Adaptive Data Rate (ADR). L'idée est de laisser le Network Server calculer la meilleure combinaison de SF / PT.

<img width="795" alt="data-rate" src="https://github.com/user-attachments/assets/5a48d735-1688-4077-a703-a8580a8abe64">

<img width="687" alt="Capture d’écran 2024-07-23 à 22 05 13" src="https://github.com/user-attachments/assets/fda81e8a-c1ba-4307-af51-f01e3077477c">

**NB : Un Device LoRaWAN doit activer le mode ADR (Flag ADR) pour utiliser cette fonctionnalité.**


