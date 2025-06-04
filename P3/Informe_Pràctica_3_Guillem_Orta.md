# Informe pràctica 3

## Part A generació d'una paguna web

### Apartat A.1

``` cpp

#include <WiFi.h>
#include <WebServer.h>

// Credencials de la xarxa Wi-Fi
const char* ssid = "*****";     // Nom de la xarxa (SSID)
const char* password = "*****"; // Contrasenya de la xarxa
WebServer server(80);           // Creem servidor web al pin 80 (HTTP)

void setup() {
  Serial.begin(115200);
  Serial.println("Intentant connectar a ");
  Serial.println(ssid);
  
  // Iniciem la connexió Wi-Fi
  WiFi.begin(ssid, password);
  
  // Esperem fins que la connexió es completi
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  
  // Quan connecta, mostrem informació per serial
  Serial.println("");
  Serial.println("Connexió WiFi exitosa");
  Serial.print("IP assignada: ");
  Serial.println(WiFi.localIP());  // Mostra l'adreça IP de l'ESP32
  
  server.on("/", handle_root);     // Configurem la funció per a la ruta arrel
  server.begin();                  // Iniciem el servidor web
  Serial.println("Servidor HTTP iniciat");
  delay(100);
}

// Definim el contingut HTML de la pàgina web
String HTML = "<!DOCTYPE html>\
<html>\
<body>\
<h1>La meva primera pàgina amb ESP32 - Mode Estació &#128522;</h1>\
</body>\
</html>";

// Funció que s'executa quan es visita la ruta arrel (/)
void handle_root() {
  server.send(200, "text/html", HTML);  // Enviem l'HTML al client
}

// NOTA: Falta la funció loop() per gestionar peticions contínuament
// Caldria afegir: void loop() { server.handleClient(); }


```

### Funcionament del codi:

El proposit del codi és, crear un servidor web bàsic en ESP32 que
es connecta a una xarxa WiFi com a estació .
Serveix una pàgina HTML simple a l'arrel (/) i mostra informació de depuració pel port sèrie.


#### Configuració inicial 

Les primeres linies de codi defineixen les credencials d'accés a la xarxa Wi-Fi i crea un objecte servidor web al pin 80.

Despres, trobem la funció setup, aquesta inicialitza la comunicació sèrie per a depuració i  intenta connectar-se a la xarxa Wi-Fi especificada.
Un cop connectat, mostra la IP assignada via sèrie i inicia el servidor web.
Per últim, la funció 'handle_root()' s'executa quan algú accedix a la ruta arrel del servidor creat.


### Apartat A.2

``` cpp
 
// webpage.h - Contingut HTML de la pàgina web
const char MAIN_page[] PROGMEM = R"=====(
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ESP32 Web Server</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            margin: 40px;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
        }
        .emoji {
            font-size: 1.5em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🏠 Pàgina Web ESP32 🚀</h1>
        <p>Benvingut/da al servidor web de l'ESP32!</p>
        <p class="emoji">⭐ Mode Estació ⭐</p>
        <p>Visites: <span id="counter">0</span></p>
    </div>
</body>
</html>
)=====";

#include <WiFi.h>
#include <WebServer.h>
#include "webpage.h"  // Incluïm el nou fitxer HTML

const char* ssid = "*****";
const char* password = "*****";
WebServer server(80);

void handle_root() {
  server.send(200, "text/html", MAIN_page);  // Usem el contingut del fitxer HTML
}

void setup() {
  Serial.begin(115200);
  
  // Connexió WiFi (igual que abans)
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nConnectat a WiFi!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());

  server.on("/", handle_root);
  server.begin();
}

void loop() {
  server.handleClient();  // Necessari per gestionar les peticions
}

```

### Explicació del codi:

Hem creat un fitxer independent (webpage.h) que només conté el codi HTML/CSS i utilitzat PROGMEM per emmagatzemar l'HTML a la memòria flash (important per a grans pàgines)



## Part B comunicació bluetooth amb el movil

```cpp 
 // Aquest codi és de domini públic (o sota llicència CC0, segons elecció)
// Autor: Evandro Copercini - 2018
//
// Aquest exemple crea un pont entre el port Sèrie i Bluetooth Clàssic (SPP)
// i demostra que SerialBT té les mateixes funcionalitats que un port Sèrie normal

#include "BluetoothSerial.h"  // Inclou la llibreria per Bluetooth Clàssic

// Comprovació de configuració del Bluetooth
#if !defined(CONFIG_BT_ENABLED) || !defined(CONFIG_BLUEDROID_ENABLED)
#error El Bluetooth no està habilitat! Executa 'make menuconfig' per activar-lo
#endif

BluetoothSerial SerialBT;  // Crea un objecte BluetoothSerial

void setup() {
  Serial.begin(115200);  // Inicialitza comunicació sèrie a 115200 bauds
  SerialBT.begin("ESP32test");  // Inicialitza Bluetooth amb nom del dispositiu
  Serial.println("Dispositiu inicialitzat. Ara pots aparellar-lo via Bluetooth!");
}

void loop() {
  // Pont de comunicació bidireccional:
  if (Serial.available()) {       // Si hi ha dades per port sèrie
    SerialBT.write(Serial.read());  // Envia les dades via Bluetooth
  }
  
  if (SerialBT.available()) {     // Si hi ha dades rebudes per Bluetooth
    Serial.write(SerialBT.read()); // Envia les dades al port sèrie
  }
  delay(20);  // Petita pausa per estabilitat
}

```

### Funcionament del codi:

Crea un pont de comunicació bidireccional entre el port sèrie UART (USB) de l'ESP32 i el port bluetooth Clàssic (SPP - Serial Port Profile).
L'ESP32 actua com a dispositiu Bluetooth amb nom "ESP32test". Qualsevol dada rebuda per una interfície es retransmet automàticament a l'altra, permetent enviar comandaments des d'un ordinador/mòbil via Bluetooth i veure'ls al Monitor Sèrie i enviar dades des del Monitor Sèrie i rebre'les en un dispositiu Bluetooth connectat.


## Parta pujar nota

### Apartat 1

``` cpp
#include <WiFi.h>
#include <WebServer.h>
#include "webpage.h"  // Inclou el fitxer HTML

WebServer server(80);

// Credencials per al punt d'accés
const char* ssid = "ESP32_AP";
const char* password = "123456789";  // Mínim 8 caràcters

void handle_root() {
  server.send(200, "text/html", MAIN_page);
}

void setup() {
  Serial.begin(115200);
  
  // Configuració del mode AP
  WiFi.softAP(ssid, password);
  
  // Mostra informació per serial
  Serial.print("Xarxa creada: ");
  Serial.println(ssid);
  Serial.print("IP de l'AP: ");
  Serial.println(WiFi.softAPIP());  // Mostra la IP de l'AP
  
  server.on("/", handle_root);
  server.begin();
  Serial.println("Servidor HTTP iniciat");
}

void loop() {
  server.handleClient();
}

const char MAIN_page[] PROGMEM = R"=====(
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>ESP32 AP Mode</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #e0f7fa;
      text-align: center;
      padding: 50px;
    }
    .container {
      background-color: white;
      padding: 30px;
      border-radius: 15px;
      box-shadow: 0 0 20px rgba(0,150,136,0.3);
      display: inline-block;
    }
    h1 {
      color: #00796b;
    }
    .ip {
      color: #00897b;
      font-size: 1.2em;
      margin: 20px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🏭 Mode Access Point Activat 🛰️</h1>
    <p>Connectat a la xarxa: <strong>ESP32_AP</strong></p>
    <p class="ip">IP del servidor: 192.168.4.1</p>
  </div>
</body>
</html>
)=====";

``` 

#### Explicació del codi:
 
#####  Canvis respete l'anterior apartat

######  Configuració Wi-Fi:

En l'apartat anterior, hem fet servir conexió STA que es definia com:

`WiFi.begin(ssid_existent, password_existent);`

Ara ho hem canviat per:

` WiFi.softAP(ssid_nou, password_nou);`

També hem canviat l'adreça IP, per la conexió STA s'obtenia la IP dell router ( Wi-Fi.localIP()), ara amb la conexió AP, s'assigna una IP fixa, concretament una amb el número '192.168.4.1'.

LA sortida pel monitor sèrie, és:

```
[Inici] Xarxa creada: ESP32_AP
[Configuració] IP de l'AP: 192.168.4.1
[Servidor] Servidor HTTP iniciat
[Connexió] Estació connectada: MAC (client) 
```

