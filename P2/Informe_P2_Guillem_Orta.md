# Informe pràctica 2

## Informe part A:

En aquesta part de la pràctica, es fa servir el codi següent:

``` c++
// Definició d'una tupla per representar el boto
struct Button {
    const uint8_t PIN;            // Assignació del pin GPIO al que va connectat el botó
    uint32_t numberKeyPresses;    // Contador de vegades que s'ha pressionat el botó
    bool pressed;                 // Variable que indica si el botó esta premut  
};

// Creació d'una variable del tipus 'Button' definida anteriorment per conectar el botó al pin 18 i inicialitza el marcatge com a false i el nombre de vegades a 0
Button button1 = {18, 0, false};

void IRAM_ATTR isr() {    
    button1.numberKeyPresses += 1;  // Increment del contador de pulsacions
    button1.pressed = true;         // Marcar el boto com a premut
}

void setup() {
    Serial.begin(115200);                          // Inicialització la comunicació serial a 115200 
    pinMode(button1.PIN, INPUT_PULLUP);            // Associem la interrupció al pin del botó, que es dispararà en el flanc de baixada (FALLING)
    attachInterrupt(button1.PIN, isr, FALLING);    
}

void loop() {
    if (button1.pressed) {                                                                   // Verifiquem si el botó ha estat premut
        Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);     // Imprimim el nombre de vegades que s'ha premut el botó
        button1.pressed = false;                                                             // Tornem a marcar que el botó no està pressionat     
    }


    static uint32_t lastMillis = 0;                                                          // Variable per emmagatzemar l'últim temps registrat
    if (millis() - lastMillis > 60000) {                                                     // Comprobar si han passat 60000 milisegons ( 1 minut)   
        lastMillis = millis();                                                               // Actualitzar l'ultim temps regustat   
        detachInterrupt(button1.PIN);                                                           
        Serial.println("Interrupt Detached!");                                               // Imprimim un missatge indicant que la interrupció s'ha 
    }
}
```
### Funcionament del codi:

#### Configuració del botó i la interrupció:

A la funció setup(), es configura el pin GPIO 18 com una entrada amb resistència pull-up interna.

Després, s'associa una interrupció a aquest pin utilitzant la funció attachInterrupt(). La interrupció es dispararà en el flanc de baixada (FALLING), és a dir, quan es premi el botó i el pin canviï de HIGH a LOW.

#### Interrupció:

Quan es prem el botó, s'executa la funció isr(), que incrementa el comptador de pulsacions (numberKeyPresses) i marca la variable 'pressed' com a 'true'.

#### Bucle principal (loop):

Al bucle principal, es verifica si la variable pressed és true. Si és així, s'imprimeix al monitor sèrie el nombre de vegades que s'ha premut el botó i es reinicia la variable pressed a false.

Després d'1 minut, la interrupció es desactiva utilitzant detachInterrupt(), i s'imprimeix un missatge al monitor  indicant que la interrupció ha estat desvinculada.


## Informe part B

``` c++

volatile int interruptCounter;  // Comptador d'interrupcions 
int totalInterruptCounter;      // Comptador total d'interrupcions

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);  // Entram en una secció crítica per protegir la variable
    interruptCounter++;                 // Incrementam el comptador d'interrupcions
    portEXIT_CRITICAL_ISR(&timerMux);  // Sortim de la secció crítica
}

void setup() {
    Serial.begin(115200);  // Inicialitzem la comunicació sèrie a 115200 

timer = timerBegin(0, 80, true);  // Inicialitzam el temporitzador 0 amb un preescalador de 80
    timerAttachInterrupt(timer, &onTimer, true);  // Associam la ISR al temporitzador
    timerAlarmWrite(timer, 1000000, true);  // Configurem l'alarma per a 1,000,000 microsegons (1 segon)
    timerAlarmEnable(timer);  // Habilitam l'alarma del temporitzador
}

void loop() {
    // Verifiquem si ha ocorregut una interrupció
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);  // Entram en una secció crítica per protegir la variable
        interruptCounter--;            // Decrementam el comptador d'interrupcions
        portEXIT_CRITICAL(&timerMux); // Sortim de la secció crítica

        totalInterruptCounter++;  // Incrementam el comptador total d'interrupcions

        // Imprimim el nombre total d'interrupcions que han ocorregut
        Serial.print("An interrupt as occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}
```
### Funcionament del codi:

#### Configuració del temporitzador:

A la funció setup(), s'inicialitza un temporitzador utilitzant timerBegin().S'associa una interrupció al temporitzador utilitzant timerAttachInterrupt(). La funció onTimer() s'executarà cada vegada que el temporitzador generi una interrupció.

El temporitzador es configura per generar una interrupció cada 1,000,000 microsegons (1 segon) utilitzant timerAlarmWrite(), i s'habilita l'alarma amb timerAlarmEnable().

#### Interrupció:
Cada vegada que el temporitzador genera una interrupció, s'executa la funció onTimer(), que incrementa la variable interruptCounter.

#### Bucle principal (loop):

Al bucle principal, es verifica si la variable interruptCounter és més gran que 0. Si és així, es decrementa interruptCounter i s'incrementa totalInterruptCounter.

Després, s'imprimeix al monitor sèrie el nombre total d'interrupcions que han ocorregut.


## Informe subir nota

``` c++
#include <Arduino.h>

// Definim els pins
const int ledPin = 2;          // Pin del LED
const int buttonUpPin = 4;     // Pin del pulsador per augmentar la freqüència
const int buttonDownPin = 5;   // Pin del pulsador per disminuir la freqüència

// Variables per controlar la freqüència de parpadeig
volatile int blinkInterval = 500;  // Interval de parpadeig inicial (500 ms)
volatile bool ledState = LOW;      // Estat del LED

// Variables per gestionar els pulsadors
volatile bool buttonUpPressed = false;
volatile bool buttonDownPressed = false;

// Temporitzador
hw_timer_t *timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

// Funció per gestionar el parpadeig del LED
void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  ledState = !ledState;  // Canviem l'estat del LED
  digitalWrite(ledPin, ledState);
  portEXIT_CRITICAL_ISR(&timerMux);
}

// Funció per llegir els pulsadors i filtrar els rebots
void IRAM_ATTR checkButtons() {
  static unsigned long lastDebounceTime = 0;
  const unsigned long debounceDelay = 50;  // Temps de debounce (50 ms)

  if (millis() - lastDebounceTime > debounceDelay) {
    if (digitalRead(buttonUpPin) == LOW) {
      buttonUpPressed = true;
    }
    if (digitalRead(buttonDownPin) == LOW) {
      buttonDownPressed = true;
    }
    lastDebounceTime = millis();
  }
}

void setup() {
  // Configuració dels pins
  pinMode(ledPin, OUTPUT);
  pinMode(buttonUpPin, INPUT_PULLUP);
  pinMode(buttonDownPin, INPUT_PULLUP);

  // Inicialització del temporitzador
  timer = timerBegin(0, 80, true);  // Temporitzador 0, preescalador de 80 (1 MHz)
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, blinkInterval * 1000, true);  // Interval en microsegons
  timerAlarmEnable(timer);

  // Configuració de la interrupció per llegir els pulsadors
  attachInterrupt(digitalPinToInterrupt(buttonUpPin), checkButtons, FALLING);
  attachInterrupt(digitalPinToInterrupt(buttonDownPin), checkButtons, FALLING);

  Serial.begin(115200);
}

void loop() {
  // Verifiquem si s'ha premut algun dels pulsadors
  if (buttonUpPressed) {
    portENTER_CRITICAL(&timerMux);
    blinkInterval -= 100;  // Disminuïm l'interval de parpadeig (augmentem la freqüència)
    if (blinkInterval < 100) blinkInterval = 100;  // Limit mínim
    timerAlarmWrite(timer, blinkInterval * 1000, true);  // Actualitzem el temporitzador
    portEXIT_CRITICAL(&timerMux);
    buttonUpPressed = false;
    Serial.print("Freqüència augmentada. Nou interval: ");
    Serial.println(blinkInterval);
  }

  if (buttonDownPressed) {
    portENTER_CRITICAL(&timerMux);
    blinkInterval += 100;  // Augmentem l'interval de parpadeig (disminuïm la freqüència)
    if (blinkInterval > 1000) blinkInterval = 1000;  // Limit màxim
    timerAlarmWrite(timer, blinkInterval * 1000, true);  // Actualitzem el temporitzador
    portEXIT_CRITICAL(&timerMux);
    buttonDownPressed = false;
    Serial.print("Freqüència disminuïda. Nou interval: ");
    Serial.println(blinkInterval);
  }
}
```

### Funcionament del codi:

#### Configuració de pins

ledPin: Pin on està connectat el LED.
buttonUpPin: Pin del pulsador per augmentar la freqüència de parpadeig.
buttonDownPin: Pin del pulsador per disminuir la freqüència de parpadeig.

#### Variables globals:

blinkInterval: Defineix l'interval de parpadeig del LED en mil·lisegons.

ledState: Estat actual del LED (HIGH o LOW).

buttonUpPressed i buttonDownPressed: Indiquen si s'ha premut algun dels pulsadors.

#### Temporitzador:

S'utilitza un temporitzador per generar interrupcions periòdiques i controlar el parpadeig del LED.

La funció onTimer() canvia l'estat del LED cada vegada que es dispara la interrupció del temporitzador.

#### Filtre de rebots:

La funció checkButtons() s'executa quan es prem un pulsador i implementa un filtre de rebots utilitzant un retard de 50 ms.

#### Interrupcions dels pulsadors:

S'utilitzen interrupcions per detectar quan es prem un pulsador. Això permet una resposta ràpida i eficient.

#### Bucle principal (loop):

Es verifica si s'ha premut algun dels pulsadors i s'ajusta l'interval de parpadeig del LED en conseqüència.

Es garanteix que l'interval de parpadeig estigui dins d'un rang raonable (entre 100 ms i 1000 ms).