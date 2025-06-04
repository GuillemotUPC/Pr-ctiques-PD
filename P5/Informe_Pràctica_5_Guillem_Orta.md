# Informe pràctica 4

## Informe exercici pràctic 1

``` cpp

void setup() {
  Serial.begin(115200);
  
  // Crear una nova tasca
  xTaskCreate(
    anotherTask,    // Funció de la tasca
    "another Task", // Nom de la tasca (per depuració)
    10000,          // Mida de la pila (en bytes)
    NULL,           // Paràmetres per passar a la tasca
    1,              // Prioritat (0-24, on 1 és baixa)
    NULL            // Handle de la tasca (per referència)
  );
}

// El bucle loop() s'executa com una tasca per defecte a FreeRTOS
void loop() {
  Serial.println("this is ESP32 Task");
  delay(1000); // Delay bloquejant de 1 segon
}

// Funció associada a la tasca "anotherTask"
void anotherTask(void *parameter) {
  // Bucle infinit
  for (;;) {
    Serial.println("this is another Task");
    delay(1000); // Delay bloquejant de 1 segon
  }
  
  // Aquesta línia mai s'executa (bucle infinit)
  vTaskDelete(NULL); // Elimina la tasca actual (teòricament)
}

```

El programa de l’Exercici Pràctic 1 genera dues tasques que s’executen de manera simultànea al microcontrolador ESP32. 
La primera tasca correspon al bucle principal loop(), que imprimeix pel port sèrie el missatge "this is ESP32 Task" cada segon utilitzant la funció delay(1000).
 La segona tasca, creada amb xTaskCreate(), executa la funció anotherTask(), que també imprimeix "this is another Task" cada segon. Com que les dues tasques tenen la mateixa prioritat i utilitzen funcions bloquejants delay(), el planificador de FreeRTOS alterna l’execució entre elles. Això produeix una sortida al port sèrie on es veuen alternats els dos missatges, com:

 
``` cpp
this is ESP32 Task  
this is another Task  
this is ESP32 Task  
...  
```

## Informe exercici pràctic 2

### Codi generat

``` cpp
#include <Arduino.h>  
#include <FreeRTOS.h>  
#include <semphr.h>  

const int ledPin = 2;  
SemaphoreHandle_t semafor;  

// Tasca per encendre el LED  
void taskEncendre(void *pvParameters) {  
    for (;;) {  
        xSemaphoreTake(semafor, portMAX_DELAY); // Espera el semàfor  
        digitalWrite(ledPin, HIGH);  
        Serial.println("LED ENCÈS");  
        vTaskDelay(1000 / portTICK_PERIOD_MS);  
        xSemaphoreGive(semafor); // Allibera el semàfor  
        vTaskDelay(10); // Permet canviar de tasca  
    }  
}  

// Tasca per apagar el LED  
void taskApagar(void *pvParameters) {  
    for (;;) {  
        xSemaphoreTake(semafor, portMAX_DELAY); // Espera el semàfor  
        digitalWrite(ledPin, LOW);  
        Serial.println("LED APAGAT");  
        vTaskDelay(1000 / portTICK_PERIOD_MS);  
        xSemaphoreGive(semafor); // Allibera el semàfor  
        vTaskDelay(10);  
    }  
}  

void setup() {  
    Serial.begin(115200);  
    pinMode(ledPin, OUTPUT);  
    semafor = xSemaphoreCreateBinary(); // Crea el semàfor binari  
    xSemaphoreGive(semafor); // Inicialitza el semàfor com alliberat  

    // Crea les tasques  
    xTaskCreate(taskEncendre, "Encendre LED", 1000, NULL, 1, NULL);  
    xTaskCreate(taskApagar, "Apagar LED", 1000, NULL, 1, NULL);  
}  

void loop() {}  

```

Per a l’Exercici Pràctic 2, es proposa un programa que sincronitza dues tasques per encendre i apagar un LED mitjançant un semàfor binari. 
El LED està connectat a un pin digital de l’ESP32, i les tasques s’encarreguen de canviar el seu estat. El semàfor binari actua com a mecanisme de control per garantir que només una tasca accedeixi al LED en cada moment.

En la inicialització (setup()), es crea el semàfor amb xSemaphoreCreateBinary(), i s’allibera inicialment amb xSemaphoreGive(). La tasca taskEncendre espera adquirir el semàfor amb xSemaphoreTake(), encén el LED, imprimeix "LED ENCÈS" al port sèrie, espera un segon amb vTaskDelay(), i allibera el semàfor abans de repetir el procés. La tasca taskApagar funciona de manera similar: adquireix el semàfor, apaga el LED, mostra "LED APAGAT", i allibera el semàfor després d’un segon. Aquesta sincronització fa que el LED alterne entre encès i apagat cada segon, mentre el port sèrie reflecteix l’ordre d’execució:


## Informe part pujar nota (rellotge)

Generarem una alarma afegint una funcionalitat.

Per implementar una alarma en el rellotge digital, es poden seguir aquests passos,primer haurem d'afegir unscomponents addicionals necessaris
Buzzer o LED addicional per indicar l’alarma (connectat a un pin GPIO) i un botó adicional per activar o desactivar l'alarma.

### Haurem de fer unes modoficacions del codi

#### Variables globals noves
``` cpp
// Variables per l'alarma  
volatile int alarm_horas = 0;  
volatile int alarm_minutos = 0;  
volatile bool alarma_activa = false;  
volatile bool alarma_sonant = false;  
```

#### Configuració de pins nous 
``` cpp

#define BUZZER 25  // Pin per al buzzer  
#define BTN_ALARMA 18 // Botó per configurar l'alarma  

void setup() {  
    // ... (codi existent)  
    pinMode(BUZZER, OUTPUT);  
    pinMode(BTN_ALARMA, INPUT_PULLUP);  
    attachInterruptArg(BTN_ALARMA, ISR_Boton, (void*)BTN_ALARMA, FALLING);  
}  
```

### Creem una nova tasca

``` cpp
 
void TareaAlarma(void *pvParameters) {  
    for (;;) {  
        if (xSemaphoreTake(relojMutex, portMAX_DELAY) == pdTRUE) {  
            // Verificar si l'alarma està activa i coincideix amb l'hora actual  
            if (alarma_activa && !alarma_sonant &&  
                horas == alarm_horas &&  
                minutos == alarm_minutos &&  
                segundos == 0) {  
                alarma_sonant = true;  
            }  

            // Activar/desactivar el buzzer mentre sona  
            if (alarma_sonant) {  
                digitalWrite(BUZZER, HIGH);  
                vTaskDelay(pdMS_TO_TICKS(500));  
                digitalWrite(BUZZER, LOW);  
                vTaskDelay(pdMS_TO_TICKS(500));  
            }  
            xSemaphoreGive(relojMutex);  
        }  
        vTaskDelay(pdMS_TO_TICKS(100)); // Revisar cada 100 ms  
    }  
}  
```

Aquest codi implementa un rellotge digital amb alarma en un ESP32 utilitzant FreeRTOS per gestionar tasques simultanees. Inclou cinc tasques principals: TareaReloj (actualitza el temps cada segon), TareaLecturaBotones (gestiona botons amb cues i anti-rebote), TareaActualizacionDisplay (mostra l’hora i l’alarma per sèrie), TareaControlLEDs (controla LEDs) i la nova TareaAlarma (activa un buzzer quan coincideix l’hora amb l’alarma). Les variables com les hores, minuts, o paràmetres de l’alarma són volatile per accés segur entre tasques, protegides per un mutex (relojMutex).

S’afegeixen un buzzer (pin 25) i un botó addicional (BTN_ALARMA) per configurar l’alarma. Prement aquest botó, es canvia entre modes d’ajust d’hores/minuts de l’alarma, mentre que BTN_INCREMENTO modifica els valors i BTN_MODO activa/desactiva l’alarma. Quan l’hora coincideix amb l’alarma activa, el buzzer sona intermitentment fins que es prem qualsevol botó.

El sistema manté l’estructura RTOS original, aprofitant cues per comunicar interrupcions de botons i semàfors per sincronitzar accés a dades. La integració de l’alarma demostra com ampliar funcionalitats de manera modular, mantenint l’eficiència amb tasques ben coordinades i temps de resposta precisos.