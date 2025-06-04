# Informe Pràctica 8

## Exercici pràctic 1

``` cpp

void setup() {
  Serial.begin(115200); // Inicialitza la comunicació (UART0) a 115200 
  Serial2.begin(9600, SERIAL_8N1, 16, 17); 
  // Inicialitza la UART2 a 9600 i assinga que el pin GPIO16 serà RXD2 d'entrada i GPIO17 serà TXD2 de sortida

  Serial.println("Inici del bucle de comunicació UART2"); 
  // Missatge de control que confirma l’inici del programa
}

void loop() {
  if (Serial.available()) { // Si hi ha dades disponibles des del terminal (UART0)
    char c = Serial.read(); // Llegeix el caràcter rebut
    Serial2.write(c);       // L’envia pel port UART2
  }

  if (Serial2.available()) { // Si hi ha dades retornades des del port UART2
    char c = Serial2.read(); // Llegeix el caràcter
    Serial.write(c);         // L’envia de nou al terminal perquè es mostri
  }
}

``` 
Aquest programa crea un bucle de comunicació entre el port UART0 
i el port UART2 de l'ESP32. 
Tot el que entra per UART0 es reenvia a UART2,i tot el que entra per UART2 es reenvia a UART0, permetent un bucle complet.

Primer, es configuren els dos ports sèrie: UART0 per comunicar-se amb el terminal de l’ordinador mitjançant USB i UART2 per gestionar la comunicació entre els pins físics GPIO16 i GPIO17. Quan l’usuari introdueix dades al monitor serial, aquestes es llegeixen per UART0 i es redirigeixen cap a UART2, on el pin de transmissió (TXD2) ha d’estar connectat físicament al pin de recepció (RXD2), creant així un bucle físic. Les dades que tornen per UART2 són llegides i reenviades novament al terminal a través de UART0, de manera que l’usuari veu per pantalla exactament el que ha enviat, validant que la comunicació funciona correctament.

## Exercici pràctic 2

``` cpp

#include <TinyGPS++.h>           // Llibreria per interpretar dades NMEA del GPS
#include <HardwareSerial.h>      // Per utilitzar un port UART addicional

TinyGPSPlus gps;                 // Creació de l'objecte GPS

HardwareSerial GPS_Serial(1);   // Definim UART1 com a port per al mòdul GPS

void setup() {
  Serial.begin(115200);         // Inicialitzem UART0 per la consola
  GPS_Serial.begin(9600, SERIAL_8N1, 4, 2); 
  // Iniciem UART1 a 9600 bauds amb RX al pin GPIO4 i TX al GPIO2
  // Connexió típica per un mòdul GPS com NEO6MV2: TX_GPS → GPIO4 (RX del ESP32)
  
  Serial.println("Inici del sistema GPS");
}

void loop() {
  while (GPS_Serial.available() > 0) {
    char c = GPS_Serial.read();  // Llegim caràcter per caràcter del GPS
    gps.encode(c);               // Els passem al parser TinyGPS++
  }

  if (gps.location.isUpdated()) {  // Si s'ha rebut una nova posició vàlida
    Serial.print("LAT: ");
    Serial.print(gps.location.lat(), 6);   // Mostrem latitud amb 6 decimals
    Serial.print(" | LON: ");
    Serial.print(gps.location.lng(), 6);   // Mostrem longitud
    Serial.print(" | SATS: ");
    Serial.print(gps.satellites.value()); // Nombre de satèl·lits visibles
    Serial.print(" | HDOP: ");
    Serial.println(gps.hdop.hdop());      // Precisió horitzontal
  }
}

``` 
Els problemes que hi havia al codi proporcionat,eren de compatibilitat amb l’ESP32, ja que aquest microcontrolador disposa de múltiples ports UART hardware i no necessita utilitzar SoftwareSerial.
Per tant, s’ha substituït l’ús de SoftwareSerial per HardwareSerial, inicialitzant l’UART1 mitjançant l’objecte HardwareSerial GPS_Serial(1), on s’han assignat els pins GPIO4 com a RX i GPIO2 com a TX. També s’ha canviat la llibreria TinyGPS per TinyGPS++, que és una versió més moderna i compatible amb l’ESP32. 

A la funció setup, es configuren els dos ports sèrie: UART0 per comunicació amb el monitor serial i UART1 per rebre dades del mòdul GPS a 9600 bauds, que és la velocitat per defecte del NEO6MV2. Al bucle principal, es llegeixen les dades NMEA enviades pel GPS caràcter a caràcter i s’envien a la llibreria. Quan es detecta una nova localització vàlida, es mostra per pantalla la latitud, longitud, el nombre de satèl·lits disponibles i la precisió (HDOP). Aquest sistema permet visualitzar dades reals del GPS en temps real a través del monitor serial. Perquè funcioni correctament, cal connectar el mòdul GPS a l'exterior, ja que de l’altra manera no es podran captar senyals dels satèl·lits. Aquesta pràctica serveix com a base per desenvolupar projectes de geolocalització amb ESP32 i sistemes embeguts.

