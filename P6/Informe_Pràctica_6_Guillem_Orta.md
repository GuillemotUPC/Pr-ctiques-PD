# Informe pràctica 6

### Exercici pràctic 1

#### Foto del montatge

![alt text](<Imagen de WhatsApp 2025-04-20 a las 21.35.11_39ada861.jpg>)

Quan s’executa el programa, el port sèrie mostra una seqüència de missatges que depenen de l’èxit o fracàs en la comunicació amb la targeta SD i la lectura de l’arxiu. 

En primer lloc, s’envia el missatge «Iniciando SD ...» per indicar l’inici del procés. Si la targeta SD s’inicialitza correctament (gràcies a la funció SD.begin(4), on el pin 4 actua com a Chip Select), es mostra «inicializacion exitosa». A continuació, s’intenta obrir l’arxiu «archivo.txt». 
Si existeix, el programa envia el missatge «archivo.txt:» i llegeix el seu contingut byte a byte, imprimint-lo pel port sèrie fins a acabar, moment en què es tanca l’arxiu. Si la targeta SD no es detecta (per exemple, per una mala connexió o format incorrecte), el port sèrie mostra «No se pudo inicializar» després del missatge inicial. 
Si l’arxiu no existeix, encara que la targeta funcioni, s’indica «Error al abrir el archivo». 

El funcionament del programa es basa en la biblioteca SPI i SD de l’ESP32: la comunicació amb la targeta es realitza mitjançant el bus SPI, configurat automàticament per la biblioteca, utilitzant el pin especificat per seleccionar la targeta. La lectura de l’arxiu es fa amb un bucle que extreu cada byte fins a acabar el contingut, assegurant-se de tancar-lo després per alliberar recursos.
El codi s’executa només una vegada , ja que el loop() no conté instruccions, i la velocitat del bus SPI, el format de les dades i la sincronització són gestionats internament per les biblioteques, sense necessitat de configuració manual per part de l’usuari.



### Exercici pràctic 2

Quan s’executa el programa del lector RFID, el port sèrie mostra inicialment el missatge «Lectura del UID», indicant que el sistema està preparat per detectar tarjetes. 

Quan s’aproxima una tarjeta RFID vàlida (com una Mifare Classic) al mòdul RC522, el programa detecta la seva presència mitjançant la funció mfrc522.PICC_IsNewCardPresent(). Si la tarjeta és nova (no s’ha llegit prèviament sense retirar-la), es llegeix el seu identificador únic (UID) amb mfrc522.PICC_ReadCardSerial(). 
El UID es mostra al port sèrie en format hexadecimal, byte per byte, amb un format com «Card UID: 3A B2 8F 1D». Si la mateixa tarjeta es manté sobre el lector sense moure’s, el programa no tornarà a detectar-la com a nova fins que es retiri i s’apropi de nou. 

El funcionament es basa en la biblioteca MFRC522, que gestiona la comunicació SPI entre l’ESP32 i el mòdul RFID: el pin 10 de l’ESP32 actua com a Slave Select (SS) per seleccionar el dispositiu, i el pin 9 com a Reset (RST). Durant la inicialització (setup()), s’activen el bus SPI i el lector RFID, mentre que al bucle principal (loop()) es verifica constantment la presència de tarjetes. Si es detecta una, es llegeix el seu UID i es mostra pel port sèrie. Si hi ha errors (com una tarjeta no compatible, connexió incorrecta dels pins o interferències), el programa no mostrarà cap dada, ja que no inclou gestió d’errors explícita. La velocitat de comunicació SPI i la configuració dels pins MOSI, MISO i SCK són gestionades automàticament per la biblioteca, simplificant la implementació.