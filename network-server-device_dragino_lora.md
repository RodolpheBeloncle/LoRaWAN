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
 
-> Une fois le device profile enregistré je vais pouvoir aller dans mon application et créer un device

NB :  Suivre le fil d'ariane au niveau de la partie headers
file d'ariane Tenants/ChirpStack/Applications/myApplication

#### 2. Ajout d'une device
Entrer un nom , une description , deviceEUI doit etre celui entré dans le device OTAA que l'on veut activer, enfin je dois lui assigner un device profile que l'on a enregistrer precedement.

Si cela n'est pas encore fait , configurer au prealable le micro controller.
Dans mon cas a moi c'est un **Arduino avec un dragino shield**
![LA668ARDUINO](https://github.com/user-attachments/assets/4711f18f-7b7c-4714-91c4-27f55139382c)

Voir doc: http://8.211.40.43:8080/xwiki/bin/view/Main/User%20Manual%20for%20LoRaWAN%20End%20Nodes/LA66%20LoRaWAN%20Shield%20User%20Manual/
Dans une console de serie (ex : arduino ide ou plateformeio )
Brancher le micro controller a la prise usb et ensuite tapper les commandes suivantes au format texte dans la console afin d'obtenir
les informations nécessaires à l'enregistrement par OTAA de la device :

EXEMPLE:

```arduino

#include <SoftwareSerial.h>

// Variables globales pour la gestion des chaînes de caractères
String inputString = "";         // une chaîne pour contenir les données entrantes
bool stringComplete = false;     // indique si la chaîne est complète

// Variables pour la gestion du temps
long old_time = millis();
long new_time;
long uplink_interval = 60000;  // 60000 ms (1 minute)

// Drapeaux pour le contrôle du flux du programme
bool time_to_at_recvb = false;
bool get_LA66_data_status = false;
bool network_joined_status = false;

// Configuration de la communication série logicielle
SoftwareSerial ss(10, 11); // Arduino RX, TX

// Tampon pour recevoir les données
char rxbuff[128];
uint8_t rxbuff_index = 0;

void setup() {
  // Initialisation des communications séries
  Serial.begin(9600);
  ss.begin(9600);
  ss.listen();
  
  inputString.reserve(200);
  
  // Réinitialisation du module LA66
  ss.println("ATZ");
}

void loop() {
  // Gestion de l'envoi périodique des données
  new_time = millis();
  if ((new_time - old_time >= uplink_interval) && (network_joined_status == 1)) {
    old_time = new_time;
    get_LA66_data_status = false;
    
    // Envoi de la chaîne "Hello World" encodée en hexadécimal
    String payload = "48656C6C6F20576F726C64"; // "Hello World" en hex
    String command = "AT+SENDB=0,2," + String(payload.length() / 2) + "," + payload;
    ss.println(command);
  }

  // Gestion de la réception des données
  if (time_to_at_recvb == true) {
    time_to_at_recvb = false;
    get_LA66_data_status = true;
    delay(1000);
    ss.println("AT+CFG");
  }
  
  // Lecture des données reçues du module LA66
  while (ss.available()) {
    char inChar = (char) ss.read();
    inputString += inChar;
    rxbuff[rxbuff_index++] = inChar;
    
    if (rxbuff_index > 128)
      break;
    
    if (inChar == '\n' || inChar == '\r') {
      stringComplete = true;
      rxbuff[rxbuff_index] = '\0';
      
      // Vérification de différents états du module
      if (strncmp(rxbuff, "JOINED", 6) == 0) {
        network_joined_status = 1;
      }
      
      if (strncmp(rxbuff, "Dragino LA66 Device", 19) == 0) {
        network_joined_status = 0;
      }
      
      if (strncmp(rxbuff, "Run AT+RECVB=? to see detail", 28) == 0) {
        time_to_at_recvb = true;
        stringComplete = false;
        inputString = "";
      }
      
      // Traitement des données reçues
      if (strncmp(rxbuff, "AT+RECVB=", 9) == 0) {
        stringComplete = false;
        inputString = "";
        Serial.print("\r\nGet downlink data(FPort & Payload) ");
        Serial.println(&rxbuff[9]);
      }
      
      rxbuff_index = 0;
      
      if (get_LA66_data_status == true) {
        stringComplete = false;
        inputString = "";
      }
    }
  }
  
  // Lecture des commandes entrées par l'utilisateur via le moniteur série
  while (Serial.available()) {
    char inChar = (char) Serial.read();
    inputString += inChar;
    if (inChar == '\n' || inChar == '\r') {
      ss.print(inputString);
      inputString = "";
    }
  }
  
  // Affichage des données reçues complètes
  if (stringComplete) {
    Serial.print(inputString);
    inputString = "";
    stringComplete = false;
  }
}

```

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

1. création de l'application
2. création de la gateway
3. création d'un device profil
4. création d'une device

#### 6. Enregistrement de la device  lorawan
Via AT COMMAND INSTALLER SUR LE FIRMWARE

Vérification des Clés
Assurez-vous que les clés configurées sur le dispositif arduino avec un shield dragino LA66 correspondent exactement à celles enregistrées dans ChirpStack.
-> soit on decide d'enregistrer d'abord sur chirspstack ou soit d'enregistrer les infos enregistrer sur chirpstack sur le stm32wioe5

-> **Exemple de Clés sur ChirpStack**
AppEUI: 52:69:73:69:6E:67:48:46
DevEUI: 2C:F7:F1:20:51:00:49:15
AppKey: (Doit être vérifiée dans l'interface ChirpStack)

-> **Clés à partir du dispositif Dragino shield LA66 Dispositif**
Normalement les identifiants vous sont livrez avec.
Vérifiez les valeurs actuelles configurées sur votre Dragino:
---
** Création de noiveau identifiants **
Configurer le DevEUI:
DevEUI = "2CF7F12051004915"

Configurer l'AppEUI:
AppEUI= "526973696E674846"

Configurer l'AppKey:
Assurez-vous que l'AppKey configuré dans le dispositif Dragino correspond exactement à celui enregistré dans ChirpStack.

APPKEY="<votre AppKey>"
Vérification et Configuration Régionale
Configurer la région:

fréquence europe =EU868
Configurer le mode OTAA:

AT+MODE=LWOTAA
Tentative de Rejoindre le Réseau

!! Voir la configuration d'une device avec le code source plus haut !!

Enfin si vous retoruner sur l'interface chirstack, cela devrait ressembler à ca :

![interface-chirpstack](https://github.com/user-attachments/assets/1dcf7cd9-c25f-449d-9b27-54c45e1791db)

La gateway et la device son activés
![Capture d’écran 2024-09-08 à 18 51 44 (2)](https://github.com/user-attachments/assets/ca79ac0c-9abe-4e8d-afd2-6bf61161c4d0)

-> **Envoie de données en queue**
![enqueur-message](https://github.com/user-attachments/assets/1b257d8b-183b-4d14-9960-8488147b43e9)
Rappel: device classe A pour que je puisse recevoir une donnée, il faut que j'emette un flux uplink.

Enqueue data : flux downlink AQ==
test : flux uplink -> AT+MSG="Data to send"

# Décripter les flux uplink et downlink
## Exemple de flux uplink
<img width="1626" alt="Capture d’écran 2024-09-12 à 21 37 14" src="https://github.com/user-attachments/assets/194df1eb-4c92-4f64-8602-9ff32c40bded">

-> Allez dans l'onglet Device profil -> Codec
<img width="1327" alt="Capture d’écran 2024-09-12 à 21 40 05" src="https://github.com/user-attachments/assets/6e7212bb-055b-424f-85f5-a03ad1e8fb79">

## Voici le code javascript code
Afin de decripter les flux uplinks et downlink

```js

// Decode uplink function.
function decodeUplink(input) {
  // Convertir les bytes en chaîne hexadécimale
  var hexString = input.bytes.map(function(byte) {
    return ('0' + (byte & 0xFF).toString(16)).slice(-2);
  }).join('');

  // Convertir la chaîne hexadécimale en texte
  var message = hexToAscii(hexString);

  return {
    data: {
      rawHex: hexString,
      decodedMessage: message
    },
    warnings: [`Decoded message: ${message}`] // Utiliser warnings pour le logging
  };
}

// Fonction pour convertir l'hexadécimal en ASCII
function hexToAscii(hex) {
  var str = '';
  for (var i = 0; i < hex.length; i += 2) {
    var charCode = parseInt(hex.substr(i, 2), 16);
    str += String.fromCharCode(charCode);
  }
  return str;
}

// Encode downlink function.
function encodeDownlink(input) {
  // Pour cet exemple, nous ne modifions pas la fonction d'encodage
  return {
    bytes: [225, 230, 255, 0]
  };
}

```

⚠️ Attention ce code est adapté pour traduire la payload du flux uplink envoyé par l'arduino


<img width="348" alt="Capture d’écran 2024-09-12 à 21 44 44" src="https://github.com/user-attachments/assets/b5bfa11a-7601-43dc-a331-0d8d55a994e1">

____
-> **Troubleshooting**
 - Vérification des ports et de l'adresse ip de l'Ébergeur du service worker
 - Activer les logs : AT + LOG=ON
 - Utilisation de l'application wirechark pour verifier les fluxs

-> **Intégration d'application via mqtt**

Modifier la configuration dans le fichier chirpstack-docker > configuration > chirpstack > chirpstack.toml
NB : Utiliser mqtt-explorer comme client
Se connecter en local 127.0.0.1:1883

<img width="844" alt="mqtt-client" src="https://github.com/user-attachments/assets/933967f2-fb63-440f-8cdc-c16c95c70b89">

1. subscribe topic uplink:
application/a52fe825-8112-401f-ae14-dffc00b55322/device/2cf7f12051004915/event/up

2. publisher topic downlink:
application/a52fe825-8112-401f-ae14-dffc00b55322/device/2cf7f12051004915/command/down

Tram a envoyer pour tester au format Json : 
{
  "confirmed": false,
  "fPort":10,
  "data":"AQ=="
}
