# Système LoRaWAN

 ## Network Server & Application Server

![Network-layout-lorawan](https://github.com/user-attachments/assets/49622f33-e5a9-474d-bd4f-a1b303eb0aae)
## Gestion de la Perméabilité des Appareils

- 🔑 **NwSKey (Authentification)** : La clé NwSKey, utilisée pour l'authentification, est gérée par le serveur réseau (Network server).
- 🔒 **AppSKey (Chiffrement)** : La clé AppSKey, utilisée pour le chiffrement des données, est gérée par le serveur d'application (Application server).

Ces pratiques assurent une séparation claire des responsabilités en matière de sécurité, améliorant ainsi la robustesse globale du système.


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


