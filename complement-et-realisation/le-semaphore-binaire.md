# Le sémaphore binaire

A partir du même montage que précédemment, analyser, comprendre et mettre en œuvre le fichier suivant qui utilise un **sémaphore binaire** et non une file d’attente.

Voir le tutoriel : [https://techtutorialsx.com/2018/09/15/esp32-arduino-getting-dht22-sensor-measurements-with-interrupts/](https://techtutorialsx.com/2018/09/15/esp32-arduino-getting-dht22-sensor-measurements-with-interrupts/)&#x20;

_Fichier : ESP32\_BinarySemaphore\_DHT22.ino_

```arduino
#include "DHTesp.h"

DHTesp dht;
hw_timer_t * timer;
SemaphoreHandle_t syncSemaphore;
int dhtPin = 4;     // Digital pin connected to the DHT22 sensor
int sampleTimer = 5 * 1000000 ; // 5 s

void IRAM_ATTR onTimer() {
  xSemaphoreGiveFromISR(syncSemaphore, NULL);
}

void setup() {
  Serial.begin(115200);
  syncSemaphore = xSemaphoreCreateBinary();
  dht.setup(dhtPin, DHTesp::DHT22);

  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, sampleTimer, true);
  timerAlarmEnable(timer);
}

void loop() {
  xSemaphoreTake(syncSemaphore, portMAX_DELAY);
  float temperature = dht.getTemperature();
  Serial.print("Temperature: ");
  Serial.println(temperature);
}
```



**Travail à faire :**

Réécrire ce programme avec 2 tâches de manière à ce que la boucle void loop() soit vide : une tâche pour gérer la partie données du DHT22 ( appelée sensorTask() ), l’autre tâche pour mettre en forme la partie Timer (appelée timerTask() ) qui est dans le setup().

Voilà la fin du code :

```arduino
void setup() {
  Serial.begin(115200);

  xTaskCreatePinnedToCore(
    sensorTask,       // Function to implement the task
    "sensorTask",     // Name of the task
    10000,            // Stack size in words
    NULL,             // Task input parameter
    1,                // Priority of the task
    NULL,             // Task handle
    0);               // Core where the task should run
  Serial.println("sensorTask created...");

  xTaskCreate(
    timerTask,       // Function to implement the task
    "timerTask",     // Name of the task
    10000,            // Stack size in words
    NULL,             // Task input parameter
    1,                // Priority of the task
    NULL);            // Task handle

  syncSemaphore = xSemaphoreCreateBinary();
  initDHT();
  void timerTask(void * pvParameters);
}

void loop() {}

```

_Voir le fichier solution : ESP32\_BinarySemaphore\_TwoTasks.ino._
