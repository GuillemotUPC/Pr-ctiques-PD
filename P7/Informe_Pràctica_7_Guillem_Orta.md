# Informe pràctica 7

## Informe pràctic part 1: Reproducció desde memoria interna

#### 1 Descripció de la sortida pel port sèrie

La sortida pel port sèrie consisteix en el missatge "Sound Generator" que s'envia cada segon quan el generador d'àudio ja no està en funcionament. Això passa  quan el fitxer d'àudio emmagatzemat a la memòria interna (PROGMEM) ha finalitzat la reproducció o si hi ha un error en la inicialització. El codi utilitza Serial.printf ("Sound Generator\n") dins del bloc else de la funció loop, que s'executa quan aac->isRunning() retorna fals.

#### Explicació del funcionament

El funcionament es basa en carregar les dades d'àudio (emmagatzemades com una matriu a sampleaac.h) mitjançant AudioFileSourcePROGMEM. La llibreria ESP8266Audio gestiona la decodificació AAC amb AudioGeneratorAAC i envia la sortida digital via I2S gràcies a AudioOutputI2S. A la funció setup, es configuren els pins I2S i el guany de sortida. Durant el loop, es processa contínuament l'àudio. 
Quan la reproducció finalitza, el generador s’atura i s’envia el missatge recurrent pel port sèrie.

#### Video del funcionament

Es pot veure a la carpeta:
https://drive.google.com/drive/folders/1iRIbj7NNO9C584HKF3zuw_RD8FgNFE1M?usp=drive_link


## Informe pràctic part 2: Reproduir un arxiu WAVE en ESP32 des d'una targeta SD externa.

#### 1. Descripció de la sortida pel port sèrie 
La sortida pel port sèrie mostra informació en temps real durant la reproducció de l'arxiu WAVE des de la targeta SD. Això inclou metadades ID3 (com títols o etiquetes), dades tècniques com el bitrate, notificacions d'estat (per exemple, "Decoding done" quan finalitza la reproducció o errors com "Failed to open file" si el fitxer no es troba), i detalls addicionals com l'URL de l'stream o el títol de la cançó. 
Aquesta informació s'envia mitjançant funcions de callback integrades a la llibreria, com audio_id3data() o audio_info(), que s'activen automàticament durant el procés de decodificació o en esdeveniments específics.

#### 2.Explicació del funcionament

El funcionament es basa en la lectura de l'arxiu d'àudio des de la targeta SD mitjançant el bus SPI. 
La llibreria ESP32-audioI2S gestiona la decodificació del fitxer WAVE i envia el flux digital al xip MAX98357A.
Durant la configuració inicial (setup()), s'estableix el volum d'àudio (amb audio.setVolume(10)) i es carrega el fitxer específic des de la SD amb audio.connecttoFS(). 
Al bucle principal (loop()), la funció audio.loop() manté el processament contínu de les dades d'àudio. Les funcions de callback, com audio_eof_mp3() o audio_bitrate(), proporcionen retroalimentació detallada pel port sèrie, permetent monitoritzar errors, el final de la reproducció o altres esdeveniments rellevants durant l'execució.